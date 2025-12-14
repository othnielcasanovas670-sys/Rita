// mobile_app/lib/services/tax_engine_service.dart
import '../models/pos_model.dart';

class TaxEngineService {
  // Налоговые ставки по классу (имитация сложной системы)
  static const Map<TaxClass, double> _taxRates = {
    TaxClass.standard: 0.10, // 10%
    TaxClass.food: 0.07,     // 7%
    TaxClass.digital: 0.05,  // 5%
    TaxClass.zeroRated: 0.0, // 0%
  };

  // Правило TAX-001: Расчет налога
  double calculateTax(List<LineItem> items, Customer? customer, String storeLocation) {
    double totalTaxableAmount = 0.0;
    
    // Если клиент освобожден от налога (например, опт)
    if (customer != null && customer.isTaxExempt) {
      return 0.0;
    }
    
    for (var item in items) {
      final rate = _taxRates[item.product.taxClass] ?? _taxRates[TaxClass.standard]!;
      
      // Налог применяется к сумме ПОСЛЕ скидок
      totalTaxableAmount += item.totalAfterDiscount * rate;
    }
    
    // В реальном приложении: корректировка ставки налога в зависимости от storeLocation
    // if (storeLocation == 'NewYork') totalTaxableAmount *= 1.01;
    
    return totalTaxableAmount;
  }
}
