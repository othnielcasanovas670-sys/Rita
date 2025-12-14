// mobile_app/lib/services/discount_rule_service.dart (Обновление)
// ... (импорты) ...

class DiscountRuleService {
  
  // ... (storeRules - остаются прежними) ...
  
  // Правило DISC-002: Генерация скидки на основе Лояльности
  Discount _getLoyaltyDiscount(Customer customer) {
    double loyaltyValue = 0.0;
    if (customer.loyaltyTier == 'Gold') loyaltyValue = 15.0; // 15%
    else if (customer.loyaltyTier == 'Silver') loyaltyValue = 10.0;
    
    return Discount(id: 'L001', name: 'Лояльность ${customer.loyaltyTier} Скидка', type: DiscountType.percentage, value: loyaltyValue, applyToItemId: '');
  }
  
  // --- Главное Улучшение: Логика Конфликта и Приоритета ---
  Map<String, List<Discount>> applyDiscounts(List<LineItem> currentItems, String? couponCode, Customer? customer) {
    final Map<String, List<Discount>> appliedMap = {};
    
    for (var item in currentItems) {
      // 1. Скидки по умолчанию
      final List<Discount> itemDiscounts = [];
      
      // 2. Применяем Лояльность (низкий приоритет, но может суммироваться)
      if (customer != null) {
        itemDiscounts.add(_getLoyaltyDiscount(customer));
      }
      
      // 3. Применяем Правила Магазина
      itemDiscounts.addAll(storeRules.where((rule) => rule.applyToItemId.isEmpty || rule.applyToItemId == item.product.sku));

      // 4. Применяем Купон (самый высокий приоритет)
      if (couponCode == 'VIP20') {
        // Правило Конфликта: Купон ИСКЛЮЧАЕТ все другие скидки, кроме фиксированных
        itemDiscounts.retainWhere((d) => d.type == DiscountType.fixedAmount); // Сохраняем только фиксированные
        itemDiscounts.add(Discount(id: 'C001', name: 'Купон VIP 20%', type: DiscountType.percentage, value: 20.0, applyToItemId: ''));
      }
      
      // 5. Оптимизация: Применить только ЛУЧШУЮ процентную скидку
      // (Если их несколько)
      
      appliedMap[item.lineId] = itemDiscounts;
    }
    return appliedMap;
  }
}
