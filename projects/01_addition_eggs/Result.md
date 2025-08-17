```
#include <stdio.h>
#include "esp_log.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

// กำหนดชื่อสำหรับแสดงใน log
static const char *TAG = "APPLES_MATH";

void app_main(void)
{
    ESP_LOGI(TAG, "🍎 เริ่มต้นโปรแกรมนับแอปเปิ้ลในตะกร้า 🍎");
    ESP_LOGI(TAG, "==========================================");
    
    // ประกาศตัวแปรเก็บจำนวนแอปเปิ้ล
    int apples_in_basket = 5;        // แอปเปิ้ลที่อยู่ในตะกร้าตั้งแต่แรก
    int apples_picked_today = 7;     // แอปเปิ้ลที่เก็บมาเพิ่มวันนี้
    int total_apples;                // แอปเปิ้ลรวมทั้งหมด
    int green_apples = 3;            // แอปเปิ้ลเขียว (อีกชนิดหนึ่ง)
    int total_all_apples;            // แอปเปิ้ลทั้งหมด (รวมทั้งแดงและเขียว)
    
    // แสดงข้อมูลเริ่มต้น
    ESP_LOGI(TAG, "📖 โจทย์:");
    ESP_LOGI(TAG, "   ในตะกร้ามีแอปเปิ้ลอยู่แล้ว: %d ผล", apples_in_basket);
    ESP_LOGI(TAG, "   วันนี้เก็บแอปเปิ้ลมาเพิ่มอีก: %d ผล", apples_picked_today);
    ESP_LOGI(TAG, "   ❓ ตอนนี้มีแอปเปิ้ลรวมกี่ผล?");
    ESP_LOGI(TAG, "");
    
    // รอสักครู่เพื่อให้อ่านโจทย์
    vTaskDelay(3000 / portTICK_PERIOD_MS);
    
    // คำนวณผลรวม (การบวก)
    total_apples = apples_in_basket + apples_picked_today;
    total_all_apples = total_apples + green_apples;

    // แสดงผลแอปเปิ้ลเขียว  
    ESP_LOGI(TAG, "🍏 และยังมีแอปเปิ้ลเขียว: %d ผล", green_apples);
    ESP_LOGI(TAG, "🍎 แอปเปิ้ลทั้งหมด (แดง+เขียว): %d ผล", total_all_apples);
    
    // แสดงขั้นตอนการคิด
    ESP_LOGI(TAG, "🧮 ขั้นตอนการคิด:");
    ESP_LOGI(TAG, "   แอปเปิ้ลที่มีอยู่ + แอปเปิ้ลที่เก็บเพิ่ม");
    ESP_LOGI(TAG, "   = %d + %d", apples_in_basket, apples_picked_today);
    ESP_LOGI(TAG, "   = %d ผล", total_apples);
    ESP_LOGI(TAG, "");
    
    // แสดงคำตอบ
    ESP_LOGI(TAG, "✅ คำตอบ:");
    ESP_LOGI(TAG, "   ตอนนี้มีแอปเปิ้ลทั้งหมด %d ผล", total_apples);
    ESP_LOGI(TAG, "");
    
    // แสดงภาพประกอบ (ASCII Art)
    ESP_LOGI(TAG, "🎨 ภาพประกอบ:");
    ESP_LOGI(TAG, "   แอปเปิ้ลเดิม: 🍎🍎🍎 (%d ผล)", apples_in_basket);
    ESP_LOGI(TAG, "   แอปเปิ้ลใหม่: 🍎🍎🍎 (%d ผล)", apples_picked_today);
    ESP_LOGI(TAG, "   รวม:        🍎🍎🍎🍎🍎🍎🍎🍎 (%d ผล)", total_apples);
    ESP_LOGI(TAG, "");
    
    // ตัวอย่างเพิ่มเติม
    ESP_LOGI(TAG, "💡 ตัวอย่างเพิ่มเติม:");
    
    // ตัวอย่างที่ 1
    int example1_old = 3;
    int example1_new = 4;
    int example1_total = example1_old + example1_new;
    ESP_LOGI(TAG, "   ถ้าในตะกร้ามีแอปเปิ้ล %d ผล และเก็บเพิ่มอีก %d ผล", example1_old, example1_new);
    ESP_LOGI(TAG, "   จะได้แอปเปิ้ลทั้งหมด %d + %d = %d ผล", example1_old, example1_new, example1_total);
    ESP_LOGI(TAG, "");
    
    // ตัวอย่างที่ 2
    int example2_old = 6;
    int example2_new = 2;
    int example2_total = example2_old + example2_new;
    ESP_LOGI(TAG, "   ถ้าในตะกร้ามีแอปเปิ้ล %d ผล และเก็บเพิ่มอีก %d ผล", example2_old, example2_new);
    ESP_LOGI(TAG, "   จะได้แอปเปิ้ลทั้งหมด %d + %d = %d ผล", example2_old, example2_new, example2_total);
    ESP_LOGI(TAG, "");
    
    // สรุปการเรียนรู้
    ESP_LOGI(TAG, "📚 สิ่งที่เรียนรู้:");
    ESP_LOGI(TAG, "   1. การบวกเลข (Addition): a + b = c");
    ESP_LOGI(TAG, "   2. การใช้ตัวแปร (Variables) เก็บค่า");
    ESP_LOGI(TAG, "   3. การแสดงผลด้วย ESP_LOGI");
    ESP_LOGI(TAG, "   4. การแก้โจทย์แบบมีขั้นตอน");
    ESP_LOGI(TAG, "");
    
    ESP_LOGI(TAG, "🎉 จบโปรแกรมนับแอปเปิ้ลในตะกร้า!");
    ESP_LOGI(TAG, "📖 อ่านต่อในโปรเจคถัดไป: 02_subtraction_oranges");
    
    // รอสักครู่ก่อนจบโปรแกรม
    vTaskDelay(2000 / portTICK_PERIOD_MS);
}
```


