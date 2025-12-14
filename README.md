// mobile_app/lib/services/inventory_manager.dart (Обновление)
// ... (импорты) ...

class InventoryManager {
  final Map<String, int> _stock = {'SKU001': 50, 'SKU002': 5, 'SKU003': 100};
  final Map<String, int> _reservedStock = {}; 

  // --- Улучшение: Метод для РЕЗЕРВИРОВАНИЯ (Optimistic Locking) ---
  // Мы резервируем в локальном чеке, полагаясь, что запасы есть.
  // Финальная проверка (Commit) произойдет при оплате.
  double reserveStock(String sku, int quantity) {
    if (!checkAvailability(sku, quantity)) {
      // Даже если нет 100% гарантии, мы позволяем добавить (с предупреждением)
      print('INVENTORY WARNING: Товар $sku может отсутствовать.');
    }
    _reservedStock[sku] = (_reservedStock[sku] ?? 0) + quantity;
    // Возвращаем текущий общий запас для записи в LineItem
    return (_stock[sku] ?? 0).toDouble(); 
  }
  
  // Правило INV-004: Финальная проверка перед списанием
  Future<bool> commitTransaction(List<LineItem> items) async {
    // В реальном приложении: отправка на бэкенд, который проверяет:
    // 1. Не изменился ли глобальный запас с момента резервирования (reservedStockAtTime).
    // 2. Достаточно ли остатков для списания.
    
    for (var item in items) {
      if (item.product.requiresInventoryTracking) {
        final currentStock = _stock[item.product.sku] ?? 0;
        if (currentStock < item.quantity) {
          print('INVENTORY ABORT: Недостаточно $item.product.sku для списания.');
          return false; // Сбой транзакции!
        }
        // Списание (имитация)
        _stock[item.product.sku] = currentStock - item.quantity;
        _reservedStock.remove(item.product.sku);
      }
    }
    print('INVENTORY: Списание успешно подтверждено (COMMIT).');
    return true;
  }
  
  bool checkAvailability(String sku, int quantityNeeded) {
    final available = (_stock[sku] ?? 0) - (_reservedStock[sku] ?? 0);
    return available >= quantityNeeded;
  }
}
