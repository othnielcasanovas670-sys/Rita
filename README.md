// mobile_app/lib/providers/cart_provider.dart (Обновление)
// ... (импорты) ...
import '../services/tax_engine_service.dart';

final taxEngineService = Provider((ref) => TaxEngineService());

class CartState {
  // ... (предыдущие поля) ...
  final Customer? currentCustomer; // Привязанный покупатель

  CartState({
    this.items = const [], 
    this.appliedCoupon,
    this.currentCustomer // Новый параметр
  });
  
  // Пересчет налогов должен быть динамическим
  double calculateTax(TaxEngineService taxService, String storeLocation) {
    return taxService.calculateTax(items, currentCustomer, storeLocation);
  }

  // Обновляем copyWith
  CartState copyWith({List<LineItem>? items, String? appliedCoupon, Customer? currentCustomer}) {
    return CartState(
      items: items ?? this.items, 
      appliedCoupon: appliedCoupon ?? this.appliedCoupon,
      currentCustomer: currentCustomer ?? this.currentCustomer,
    );
  }
}

class CartNotifier extends StateNotifier<CartState> {
  // ... (предыдущие поля) ...
  final String _storeLocation = 'KYIV-STORE-001'; // Местоположение для налога

  // ... (addItem - остается похожим, но использует Customer) ...

  void attachCustomer(Customer customer) {
    // Привязка клиента вызывает пересчет скидок (для Лояльности)
    _updateCartWithDiscounts(state.items, state.appliedCoupon, customer);
    state = state.copyWith(currentCustomer: customer);
    _ref.read(auditService).logAction(AuditAction.discountApplied, _currentUserId, 'Привязан клиент ${customer.name}');
  }
  
  // --- Обновленный метод пересчета ---
  void _updateCartWithDiscounts(List<LineItem> currentItems, String? coupon, Customer? customer) {
    final discountService = _ref.read(discountService);
    final appliedRules = discountService.applyDiscounts(currentItems, coupon, customer);
    
    final updatedItems = currentItems.map((item) {
      final discounts = appliedRules[item.lineId] ?? [];
      // В реальном коде: здесь LineItem должен быть скопирован с новыми скидками
      return LineItem(
        product: item.product,
        quantity: item.quantity,
        priceOverride: item.priceOverride,
        appliedDiscounts: discounts,
        reservedStockAtTime: item.reservedStockAtTime,
      );
    }).toList();
    
    state = state.copyWith(items: updatedItems);
  }

  // --- Финализация и оплата (Обновлено для Inventory Commit и Лояльности) ---
  Future<Transaction> finalizeTransaction(String paymentMethod) async {
    // 1. Финальная проверка и COMMIT Inventory
    final inventoryCommitSuccess = await _ref.read(inventoryManagerProvider).commitTransaction(state.items);
    if (!inventoryCommitSuccess) {
      _ref.read(auditService).logAction(AuditAction.transactionCompleted, _currentUserId, 'Транзакция ОТМЕНЕНА: Сбой инвентаризации');
      throw Exception('Ошибка: Недостаточно товара на складе.');
    }
    
    // 2. Расчет Лояльности
    final pointsEarned = state.currentCustomer != null ? (state.netTotal / 10).round() : 0; // 1 балл за каждые 10 у.е.
    final taxService = _ref.read(taxEngineService);
    final tax = taxService.calculateTax(state.items, state.currentCustomer, _storeLocation);

    final transaction = Transaction(
      transactionId: 'TX-${Random().nextInt(99999)}',
      timestamp: DateTime.now(),
      items: state.items,
      subtotal: state.subtotal,
      discountTotal: state.totalDiscount,
      taxAmount: tax,
      grandTotal: state.netTotal + tax,
      status: TransactionStatus.completed,
      cashHandlerId: _currentUserId,
      customer: state.currentCustomer,
      pointsEarned: pointsEarned.toDouble(),
      pointsRedeemed: 0, // Усложнение: здесь могла быть логика списания баллов
    );
    
    // 3. Обновление баллов клиента (имитация)
    if (state.currentCustomer != null) {
        state.currentCustomer!.copyWith(loyaltyPoints: state.currentCustomer!.loyaltyPoints + pointsEarned);
    }

    _ref.read(auditService).logAction(AuditAction.transactionCompleted, _currentUserId, 'Успешная транзакция ${transaction.transactionId}. Баллов заработано: $pointsEarned');
    state = CartState();
    return transaction;
  }
}
