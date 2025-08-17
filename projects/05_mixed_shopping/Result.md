```
#include <stdio.h>
#include <string.h>
#include "esp_log.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

static const char *TAG = "SHOPPING_MATH";

// โครงสร้างข้อมูลสินค้า
typedef struct {
    char name[40];          // ✅ เพิ่มขนาดจาก 20 เป็น 40
    int quantity;           
    float price_per_unit;
    float total_price;
} product_t;

// ฟังก์ชันคำนวณราคาสินค้า
void calculate_product_total(product_t *product) {
    product->total_price = product->quantity * product->price_per_unit;
}

// ฟังก์ชันแสดงรายการสินค้า
void display_product(const product_t *product) {
    ESP_LOGI(TAG, "   %s: %d × %.0f = %.0f บาท", 
             product->name, product->quantity, product->price_per_unit, product->total_price);
}

// ฟังก์ชันคำนวณราคารวมทั้งหมด
float calculate_total_bill(product_t products[], int count) {
    float total = 0.0;
    for (int i = 0; i < count; i++) {
        calculate_product_total(&products[i]);
        total += products[i].total_price;
    }
    return total;
}

// ฟังก์ชันใช้ส่วนลด
float apply_discount(float total, float discount) {
    return total - discount;
}

// ฟังก์ชันแบ่งจ่าย
float split_payment(float amount, int people) {
    if (people <= 0) {
        ESP_LOGE(TAG, "Error: จำนวนคนต้องมากกว่า 0");
        return 0.0;
    }
    return amount / people;
}

// ฟังก์ชันคิด VAT
float apply_vat(float amount, float vat_rate) {
    return amount * (1 + vat_rate / 100.0);
}

void app_main(void)
{
    ESP_LOGI(TAG, "🛒 เริ่มต้นโปรแกรมซื้อของที่ตลาด 🛒");
    ESP_LOGI(TAG, "=====================================");
    
    // ข้อมูลการซื้อของ
    product_t products[] = {
        {"แอปเปิ้ล", 6, 15.0, 0.0},
        {"กล้วย", 12, 8.0, 0.0},
        {"ส้ม", 8, 12.0, 0.0},
        {"ชานมไข่มุก", 2, 45.0, 0.0}
    };
    int product_count = sizeof(products) / sizeof(products[0]);

    float discount_percent = 10.0;
    float vat_rate = 7.0;
    int people = 3;

    float subtotal = calculate_total_bill(products, product_count);
    float discount_amount = subtotal * (discount_percent / 100.0);

    ESP_LOGI(TAG, "📖 โจทย์:");
    ESP_LOGI(TAG, "   แม่ไปซื้อของที่ตลาด:");
    for (int i = 0; i < product_count; i++) {
        ESP_LOGI(TAG, "   - %s: %d หน่วย หน่วยละ %.0f บาท", 
                 products[i].name, products[i].quantity, products[i].price_per_unit);
    }
    ESP_LOGI(TAG, "   - มีส่วนลด: %.2f บาท", discount_amount);
    ESP_LOGI(TAG, "   - แบ่งจ่าย: %d คน", people);
    ESP_LOGI(TAG, "");
    
    vTaskDelay(3000 / portTICK_PERIOD_MS);
    
    ESP_LOGI(TAG, "🧮 ขั้นตอนการคิด:");
    ESP_LOGI(TAG, "   1. คำนวณราคาแต่ละสินค้า (การคูณ):");

    for (int i = 0; i < product_count; i++) {
        ESP_LOGI(TAG, "      %s: %d × %.0f = %.0f บาท", 
                 products[i].name, products[i].quantity, 
                 products[i].price_per_unit, products[i].total_price);
    }
    ESP_LOGI(TAG, "");

    ESP_LOGI(TAG, "   2. รวมราคาทั้งหมด (การบวก):");
    ESP_LOGI(TAG, "      ยอดรวม: %.2f บาท", subtotal);
    ESP_LOGI(TAG, "");

    float after_discount = apply_discount(subtotal, discount_amount);
    ESP_LOGI(TAG, "   3. หักส่วนลด %.0f%%:", discount_percent);
    ESP_LOGI(TAG, "      %.2f - %.2f = %.2f บาท", subtotal, discount_amount, after_discount);

    float after_vat = apply_vat(after_discount, vat_rate);
    ESP_LOGI(TAG, "   4. คิด VAT %.0f%%:", vat_rate);
    ESP_LOGI(TAG, "      %.2f × 1.07 = %.2f บาท", after_discount, after_vat);

    float per_person = split_payment(after_vat, people);
    ESP_LOGI(TAG, "   5. แบ่งจ่าย:");
    ESP_LOGI(TAG, "      %.2f ÷ %d = %.2f บาท/คน", after_vat, people, per_person);

    ESP_LOGI(TAG, "🧾 ใบเสร็จรับเงิน:");
    ESP_LOGI(TAG, "   ==========================================");
    ESP_LOGI(TAG, "   🏪 ตลาดสดใหม่ 🏪");
    ESP_LOGI(TAG, "   ==========================================");
    for (int i = 0; i < product_count; i++) {
        display_product(&products[i]);
    }

    ESP_LOGI(TAG, "   ----------------------------------------");
    ESP_LOGI(TAG, "   ยอดรวม:                 %.2f บาท", subtotal);
    ESP_LOGI(TAG, "   ส่วนลด %.0f%%:             -%.2f บาท", discount_percent, discount_amount);
    ESP_LOGI(TAG, "   หลังหักส่วนลด:          %.2f บาท", after_discount);
    ESP_LOGI(TAG, "   ภาษี VAT %.0f%%:           +%.2f บาท", vat_rate, after_vat - after_discount);
    ESP_LOGI(TAG, "   ========================================");
    ESP_LOGI(TAG, "   ยอดสุทธิรวม VAT:        %.2f บาท", after_vat);
    ESP_LOGI(TAG, "   แบ่งจ่าย %d คน:           %.2f บาท/คน", people, per_person);
    ESP_LOGI(TAG, "   ========================================");
    ESP_LOGI(TAG, "   ขอบคุณที่ใช้บริการ 😊");
    ESP_LOGI(TAG, "");

    // ตัวอย่างเพิ่มเติม
    ESP_LOGI(TAG, "💡 ตัวอย่างเพิ่มเติม:");
    ESP_LOGI(TAG, "   📝 ถ้าเพิ่มมะม่วง 4 ผล ผลละ 25 บาท:");
    float mango_total = 4 * 25;
    float new_subtotal = subtotal + mango_total;
    float new_discount_amount = new_subtotal * (discount_percent / 100.0);
    float new_discounted = apply_discount(new_subtotal, new_discount_amount);
    float new_per_person = split_payment(new_discounted, people);
    ESP_LOGI(TAG, "      มะม่วง: 4 × 25 = %.0f บาท", mango_total);
    ESP_LOGI(TAG, "      ยอดรวมใหม่: %.0f + %.0f = %.0f บาท", subtotal, mango_total, new_subtotal);
    ESP_LOGI(TAG, "      หักส่วนลด %.0f%%: %.0f - %.2f = %.2f บาท", discount_percent, new_subtotal, new_discount_amount, new_discounted);
    ESP_LOGI(TAG, "      แบ่งจ่าย: %.2f ÷ %d = %.2f บาท/คน", new_discounted, people, new_per_person);
    ESP_LOGI(TAG, "");

    // ตัวอย่างที่ 2
    ESP_LOGI(TAG, "   🏷️ ถ้าใช้ส่วนลด 10%% แทน:");
    float percent_discount = subtotal * 0.10;
    float percent_discounted = apply_discount(subtotal, percent_discount);
    float percent_per_person = split_payment(percent_discounted, people);
    ESP_LOGI(TAG, "      ส่วนลด 10%%: %.0f × 0.10 = %.2f บาท", subtotal, percent_discount);
    ESP_LOGI(TAG, "      ยอดสุทธิ: %.0f - %.2f = %.2f บาท", subtotal, percent_discount, percent_discounted);
    ESP_LOGI(TAG, "      แบ่งจ่าย: %.2f ÷ %d = %.2f บาท/คน", percent_discounted, people, percent_per_person);

    ESP_LOGI(TAG, "🎉 จบโปรแกรมซื้อของที่ตลาด!");
    ESP_LOGI(TAG, "📖 อ่านต่อในโปรเจคถัดไป: 06_advanced_math");

    vTaskDelay(2000 / portTICK_PERIOD_MS);
}

```
