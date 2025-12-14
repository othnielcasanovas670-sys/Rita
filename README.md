// mobile_app/lib/models/pos_model.dart (Дополнение)
// ... (существующие enum, Product, Discount - остаются прежними) ...

enum TaxClass { standard, food, digital, zeroRated } // Классы налогообложения

class Product {
  // ... (предыдущие поля) ...
  final TaxClass taxClass; // Новый класс налога
  
  Product({
    // ... (предыдущие поля) ...
    this.taxClass = TaxClass.standard,
  });
}

class Customer {
  final String customerId;
  final String name;
  final String loyaltyTier; // Bronze, Silver, Gold
  final int loyaltyPoints;
  final bool isTaxExempt; // Освобождение от налога (например, оптовые покупатели)

  Customer({required this.customerId, required this.name, this.loyaltyTier = 'Bronze', this.loyaltyPoints = 0, this.isTaxExempt = false});
  
  Customer copyWith({int? loyaltyPoints}) {
    return Customer(
      customerId: customerId, 
      name: name, 
      loyaltyTier: loyaltyTier,
      loyaltyPoints: loyaltyPoints ?? this.loyaltyPoints,
      isTaxExempt: isTaxExempt,
    );
  }
}

class LineItem {
  // ... (предыдущие поля) ...
  double reservedStockAtTime; // Сколько товара было зарезервировано
  
  // Добавляем конструктор, чтобы включить новое поле
  LineItem({
    // ... (предыдущие поля) ...
    required this.reservedStockAtTime,
  }) : lineId = 'L-${Random().nextInt(99999)}';
  
  // Обновляем copyWith для LineItem (обязательно для Riverpod)
  LineItem copyWith({int? quantity, double? priceOverride, List<Discount>? appliedDiscounts}) {
    return LineItem(
        product: product, 
        quantity: quantity ?? this.quantity, 
        priceOverride: priceOverride ?? this.priceOverride, 
        appliedDiscounts: appliedDiscounts ?? this.appliedDiscounts, 
        reservedStockAtTime: reservedStockAtTime
    );
  }

  // ... (getter'ы subtotalBeforeDiscount, totalAfterDiscount) ...
}

class Transaction {
  // ... (предыдущие поля) ...
  final Customer? customer; // Привязка к покупателю
  final double pointsEarned; // Сколько баллов заработано
  final double pointsRedeemed; // Сколько баллов потрачено
  
  Transaction({
    // ... (предыдущие поля) ...
    this.customer, required this.pointsEarned, required this.pointsRedeemed
  });
}
