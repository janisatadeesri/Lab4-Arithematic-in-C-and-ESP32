```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <time.h>
#include "esp_log.h"
#include "esp_timer.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

// 🏷️ Tag สำหรับ Log
static const char *TAG = "FINAL_CALCULATOR";

// 🎯 ค่าคงที่
#define PI 3.14159265359
#define MAX_HISTORY 50
#define MAX_DISPLAY_LENGTH 20
#define VERSION "1.0.0"

// 📊 enum สำหรับโหมดการทำงาน
typedef enum {
    MODE_MAIN_MENU = 0,
    MODE_BASIC,
    MODE_ADVANCED,
    MODE_SHOP,
    MODE_HISTORY,
    MODE_EXIT
} calculator_mode_t;

// 🧮 enum สำหรับการดำเนินการ
typedef enum {
    OP_ADD = 1, OP_SUBTRACT, OP_MULTIPLY, OP_DIVIDE,
    OP_POWER, OP_SQRT, OP_FACTORIAL,
    OP_AREA_CIRCLE, OP_AREA_RECTANGLE, OP_VOLUME_BOX,
    OP_PERCENTAGE, OP_DISCOUNT, OP_TAX
} operation_t;

// 🧾 โครงสร้างประวัติการคำนวณ
typedef struct {
    int id;
    operation_t operation;
    double operand1;
    double operand2;
    double result;
    char timestamp[20];
    char description[100];
} calculation_history_t;

// 🛒 โครงสร้างสินค้า
typedef struct {
    int id;
    char name[50];
    double price;
    int quantity;
    double total;
} product_t;

// 💾 โครงสร้างข้อมูลเครื่องคิดเลข
typedef struct {
    calculation_history_t history[MAX_HISTORY];
    int history_count;
    int total_calculations;
    double total_computation_time;
    calculator_mode_t current_mode;
    product_t cart[10];
    int cart_count;
    double shop_total;
    double shop_discount;
    double shop_tax_rate;
} calculator_data_t;

// 🌍 ตัวแปรโกลบอล
static calculator_data_t calc_data = {0};

// 🎨 ฟังก์ชันแสดง ASCII Art Logo
void show_logo(void) {
    ESP_LOGI(TAG, "╔════════════════════════════════════════════════╗");
    ESP_LOGI(TAG, "║          🧮 เครื่องคิดเลขครบครัน v%s        ║", VERSION);
    ESP_LOGI(TAG, "║                ESP32 Calculator               ║");
    ESP_LOGI(TAG, "╠════════════════════════════════════════════════╣");
    ESP_LOGI(TAG, "║  📱 Modern • 🛡️ Safe • ⚡ Fast • 🎯 Accurate  ║");
    ESP_LOGI(TAG, "╚════════════════════════════════════════════════╝");
    ESP_LOGI(TAG, "");
    ESP_LOGI(TAG, "    🧮    💻    📊    🏪");
    ESP_LOGI(TAG, "   Basic Advanced Stats Shop");
    ESP_LOGI(TAG, "");
}

// ⏰ ฟังก์ชันสร้าง timestamp
void create_timestamp(char* buffer, size_t size) {
    time_t now;
    struct tm timeinfo;
    time(&now);
    localtime_r(&now, &timeinfo);
    strftime(buffer, size, "%Y-%m-%d %H:%M:%S", &timeinfo);
}

// 💾 ฟังก์ชันบันทึกประวัติ
void save_to_history(operation_t op, double op1, double op2, double result, const char* desc) {
    if (calc_data.history_count >= MAX_HISTORY) {
        // ลบประวัติเก่าที่สุด
        for (int i = 0; i < MAX_HISTORY - 1; i++) {
            calc_data.history[i] = calc_data.history[i + 1];
        }
        calc_data.history_count = MAX_HISTORY - 1;
    }
    
    calculation_history_t* entry = &calc_data.history[calc_data.history_count];
    entry->id = calc_data.total_calculations + 1;
    entry->operation = op;
    entry->operand1 = op1;
    entry->operand2 = op2;
    entry->result = result;
    strncpy(entry->description, desc, sizeof(entry->description) - 1);
    create_timestamp(entry->timestamp, sizeof(entry->timestamp));
    
    calc_data.history_count++;
    calc_data.total_calculations++;
    
    ESP_LOGI(TAG, "💾 บันทึกประวัติ #%d: %s", entry->id, desc);
}

// 🔢 ฟังก์ชันการคำนวณพื้นฐาน
double safe_add(double a, double b) {
    return a + b;
}

double safe_subtract(double a, double b) {
    return a - b;
}

double safe_multiply(double a, double b) {
    return a * b;
}

double safe_divide(double a, double b) {
    if (b == 0.0) {
        ESP_LOGE(TAG, "❌ ข้อผิดพลาด: ไม่สามารถหารด้วยศูนย์ได้!");
        return NAN;
    }
    return a / b;
}

double safe_power(double base, double exponent) {
    if (base == 0.0 && exponent < 0) {
        ESP_LOGE(TAG, "❌ ข้อผิดพลาด: 0 ยกกำลังลบไม่ได้!");
        return NAN;
    }
    return pow(base, exponent);
}

double safe_sqrt(double a) {
    if (a < 0) {
        ESP_LOGE(TAG, "❌ ข้อผิดพลาด: ไม่สามารถหารากที่สองของจำนวนลบได้!");
        return NAN;
    }
    return sqrt(a);
}

double safe_factorial(int n) {
    if (n < 0) {
        ESP_LOGE(TAG, "❌ ข้อผิดพลาด: แฟกทอเรียลของจำนวนลบไม่ได้!");
        return NAN;
    }
    if (n > 20) {
        ESP_LOGW(TAG, "⚠️ เตือน: แฟกทอเรียลใหญ่เกินไป!");
        return INFINITY;
    }
    
    double result = 1.0;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

// 📐 ฟังก์ชันเรขาคณิต
double calculate_circle_area(double radius) {
    if (radius < 0) {
        ESP_LOGE(TAG, "❌ รัศมีไม่สามารถเป็นลบได้!");
        return NAN;
    }
    return PI * radius * radius;
}

double calculate_rectangle_area(double length, double width) {
    if (length < 0 || width < 0) {
        ESP_LOGE(TAG, "❌ ความยาวและความกว้างไม่สามารถเป็นลบได้!");
        return NAN;
    }
    return length * width;
}

double calculate_box_volume(double length, double width, double height) {
    if (length < 0 || width < 0 || height < 0) {
        ESP_LOGE(TAG, "❌ ขนาดทุกด้านต้องเป็นบวก!");
        return NAN;
    }
    return length * width * height;
}

// 💰 ฟังก์ชันการเงิน
double calculate_percentage(double value, double percent) {
    return (value * percent) / 100.0;
}

double apply_discount(double original_price, double discount_percent) {
    if (discount_percent < 0 || discount_percent > 100) {
        ESP_LOGW(TAG, "⚠️ ส่วนลดควรอยู่ระหว่าง 0-100%%");
        return original_price;
    }
    double discount = calculate_percentage(original_price, discount_percent);
    return original_price - discount;
}

double apply_tax(double amount, double tax_rate) {
    if (tax_rate < 0) {
        ESP_LOGW(TAG, "⚠️ อัตราภาษีไม่ควรเป็นลบ");
        return amount;
    }
    double tax = calculate_percentage(amount, tax_rate);
    return amount + tax;
}

// 🎯 ฟังก์ชันประมวลผลการคำนวณ
double perform_calculation(operation_t op, double op1, double op2) {
    int64_t start_time = esp_timer_get_time();
    double result = 0.0;
    char description[100];
    
    switch (op) {
        case OP_ADD:
            result = safe_add(op1, op2);
            sprintf(description, "%.2f + %.2f = %.2f", op1, op2, result);
            break;
        case OP_SUBTRACT:
            result = safe_subtract(op1, op2);
            sprintf(description, "%.2f - %.2f = %.2f", op1, op2, result);
            break;
        case OP_MULTIPLY:
            result = safe_multiply(op1, op2);
            sprintf(description, "%.2f × %.2f = %.2f", op1, op2, result);
            break;
        case OP_DIVIDE:
            result = safe_divide(op1, op2);
            sprintf(description, "%.2f ÷ %.2f = %.2f", op1, op2, result);
            break;
        case OP_POWER:
            result = safe_power(op1, op2);
            sprintf(description, "%.2f ^ %.2f = %.2f", op1, op2, result);
            break;
        case OP_SQRT:
            result = safe_sqrt(op1);
            sprintf(description, "√%.2f = %.2f", op1, result);
            break;
        case OP_FACTORIAL:
            result = safe_factorial((int)op1);
            sprintf(description, "%.0f! = %.0f", op1, result);
            break;
        case OP_AREA_CIRCLE:
            result = calculate_circle_area(op1);
            sprintf(description, "พื้นที่วงกลม r=%.2f = %.2f", op1, result);
            break;
        case OP_AREA_RECTANGLE:
            result = calculate_rectangle_area(op1, op2);
            sprintf(description, "พื้นที่สี่เหลี่ยม %.2f×%.2f = %.2f", op1, op2, result);
            break;
        case OP_VOLUME_BOX:
            // ใช้ op1 เป็น length×width, op2 เป็น height
            result = op1 * op2;
            sprintf(description, "ปริมาตรกล่อง = %.2f", result);
            break;
        case OP_PERCENTAGE:
            result = calculate_percentage(op1, op2);
            sprintf(description, "%.2f%% ของ %.2f = %.2f", op2, op1, result);
            break;
        case OP_DISCOUNT:
            result = apply_discount(op1, op2);
            sprintf(description, "ลด %.2f%% จาก %.2f = %.2f", op2, op1, result);
            break;
        case OP_TAX:
            result = apply_tax(op1, op2);
            sprintf(description, "ภาษี %.2f%% จาก %.2f = %.2f", op2, op1, result);
            break;
        default:
            ESP_LOGE(TAG, "❌ การดำเนินการไม่รู้จัก!");
            return NAN;
    }
    
    int64_t end_time = esp_timer_get_time();
    double computation_time = (end_time - start_time) / 1000.0; // มิลลิวินาที
    calc_data.total_computation_time += computation_time;
    
    if (!isnan(result) && !isinf(result)) {
        save_to_history(op, op1, op2, result, description);
        ESP_LOGI(TAG, "✅ %s", description);
        ESP_LOGI(TAG, "⏱️ ใช้เวลา: %.3f มิลลิวินาที", computation_time);
    }
    
    return result;
}

// 🖥️ ฟังก์ชันแสดงเมนูหลัก
void show_main_menu(void) {
    ESP_LOGI(TAG, "\n╔══════════════════════════════════════════════════╗");
    ESP_LOGI(TAG, "║                   🧮 เมนูหลัก                   ║");
    ESP_LOGI(TAG, "╠══════════════════════════════════════════════════╣");
    ESP_LOGI(TAG, "║ [1] 🔢 โหมดพื้นฐาน - Basic Calculator         ║");
    ESP_LOGI(TAG, "║ [2] 🔬 โหมดขั้นสูง - Advanced Mathematics     ║");
    ESP_LOGI(TAG, "║ [3] 🏪 โหมดร้านค้า - Shop POS System          ║");
    ESP_LOGI(TAG, "║ [4] 📊 โหมดประวัติ - History & Statistics     ║");
    ESP_LOGI(TAG, "║ [0] 🚪 ออกจากโปรแกรม - Exit                  ║");
    ESP_LOGI(TAG, "╚══════════════════════════════════════════════════╝");
    ESP_LOGI(TAG, "");
    ESP_LOGI(TAG, "📊 สถิติ: %d การคำนวณ | %.2f มิลลิวินาที รวม", 
             calc_data.total_calculations, calc_data.total_computation_time);
}

// 🔢 ฟังก์ชันโหมดพื้นฐาน
void basic_calculator_mode(void) {
    ESP_LOGI(TAG, "\n🔢 === โหมดพื้นฐาน ===");
    ESP_LOGI(TAG, "╔═══════════════════════════════════════╗");
    ESP_LOGI(TAG, "║         การดำเนินการพื้นฐาน         ║");
    ESP_LOGI(TAG, "╠═══════════════════════════════════════╣");
    ESP_LOGI(TAG, "║ [1] ➕ บวก     [2] ➖ ลบ            ║");
    ESP_LOGI(TAG, "║ [3] ✖️ คูณ      [4] ➗ หาร           ║");
    ESP_LOGI(TAG, "║ [5] 🔢 ยกกำลัง [6] √ รากที่สอง      ║");
    ESP_LOGI(TAG, "║ [7] ! แฟกทอเรียล                    ║");
    ESP_LOGI(TAG, "╚═══════════════════════════════════════╝");
    
    // จำลองการเลือกและการคำนวณ
    double demo_values[][2] = {
        {25.5, 14.3},    // บวก
        {100.0, 37.5},   // ลบ
        {12.0, 8.0},     // คูณ
        {144.0, 12.0},   // หาร
        {2.0, 8.0},      // ยกกำลัง
        {64.0, 0},       // รากที่สอง
        {5.0, 0}         // แฟกทอเรียล
    };
    
    operation_t operations[] = {OP_ADD, OP_SUBTRACT, OP_MULTIPLY, OP_DIVIDE, 
                               OP_POWER, OP_SQRT, OP_FACTORIAL};
    
    for (int i = 0; i < 7; i++) {
        vTaskDelay(pdMS_TO_TICKS(1500));
        ESP_LOGI(TAG, "\n🎯 ตัวอย่างที่ %d:", i + 1);
        perform_calculation(operations[i], demo_values[i][0], demo_values[i][1]);
    }
}

// 🔬 ฟังก์ชันโหมดขั้นสูง
void advanced_calculator_mode(void) {
    ESP_LOGI(TAG, "\n🔬 === โหมดขั้นสูง ===");
    ESP_LOGI(TAG, "╔══════════════════════════════════════════╗");
    ESP_LOGI(TAG, "║            คณิตศาสตร์ขั้นสูง           ║");
    ESP_LOGI(TAG, "╠══════════════════════════════════════════╣");
    ESP_LOGI(TAG, "║ 📐 เรขาคณิต และ การคำนวณพิเศษ         ║");
    ESP_LOGI(TAG, "╚══════════════════════════════════════════╝");
    
    // ตัวอย่างการคำนวณขั้นสูง
    vTaskDelay(pdMS_TO_TICKS(1000));
    ESP_LOGI(TAG, "\n🎯 พื้นที่วงกลม รัศมี 5 เมตร:");
    perform_calculation(OP_AREA_CIRCLE, 5.0, 0);
    
    vTaskDelay(pdMS_TO_TICKS(1000));
    ESP_LOGI(TAG, "\n🎯 พื้นที่สี่เหลี่ยม 8×6 เมตร:");
    perform_calculation(OP_AREA_RECTANGLE, 8.0, 6.0);
    
    vTaskDelay(pdMS_TO_TICKS(1000));
    ESP_LOGI(TAG, "\n🎯 15% ของ 200 บาท:");
    perform_calculation(OP_PERCENTAGE, 200.0, 15.0);
}

// 🏪 ฟังก์ชันโหมดร้านค้า
void shop_mode(void) {
    ESP_LOGI(TAG, "\n🏪 === โหมดร้านค้า ===");
    ESP_LOGI(TAG, "🛒 ระบบ POS ร้านสะดวกซื้อ \"คิดเก่ง\"");
    
    // เคลียร์ตะกร้า
    calc_data.cart_count = 0;
    calc_data.shop_total = 0;
    calc_data.shop_discount = 10.0;  // ส่วนลด 10%
    calc_data.shop_tax_rate = 7.0;   // ภาษี 7%
    
    // จำลองการเพิ่มสินค้า
    product_t demo_products[] = {
        {1, "น้ำดื่ม", 15.0, 2, 0},
        {2, "ขนมปัง", 25.0, 1, 0},
        {3, "กาแฟกระป๋อง", 45.0, 3, 0}
    };
    
    ESP_LOGI(TAG, "\n🛒 เพิ่มสินค้าในตะกร้า:");
    for (int i = 0; i < 3; i++) {
        calc_data.cart[i] = demo_products[i];
        calc_data.cart[i].total = demo_products[i].price * demo_products[i].quantity;
        calc_data.shop_total += calc_data.cart[i].total;
        calc_data.cart_count++;
        
        ESP_LOGI(TAG, "➕ %s: %.2f × %d = %.2f บาท", 
                 demo_products[i].name, demo_products[i].price, 
                 demo_products[i].quantity, calc_data.cart[i].total);
        vTaskDelay(pdMS_TO_TICKS(800));
    }
    
    ESP_LOGI(TAG, "\n💰 สรุปการคำนวณ:");
    ESP_LOGI(TAG, "╔════════════════════════════════════════════╗");
    ESP_LOGI(TAG, "║              🧾 ใบเสร็จ                  ║");
    ESP_LOGI(TAG, "╠════════════════════════════════════════════╣");
    
    for (int i = 0; i < calc_data.cart_count; i++) {
        ESP_LOGI(TAG, "║ %s  %.2f×%d  %.2f ║", 
                 calc_data.cart[i].name, calc_data.cart[i].price,
                 calc_data.cart[i].quantity, calc_data.cart[i].total);
    }
    
    ESP_LOGI(TAG, "╠════════════════════════════════════════════╣");
    ESP_LOGI(TAG, "║ 📊 ยอดรวม:                    %.2f บาท ║", calc_data.shop_total);
    
    // คำนวณส่วนลด
    double discount_amount = calculate_percentage(calc_data.shop_total, calc_data.shop_discount);
    double after_discount = calc_data.shop_total - discount_amount;
    ESP_LOGI(TAG, "║ 🎫 ส่วนลด %.0f%%:               -%.2f บาท ║", 
             calc_data.shop_discount, discount_amount);
    ESP_LOGI(TAG, "║ 💵 หลังหักส่วนลด:             %.2f บาท ║", after_discount);
    
    // คำนวณภาษี
    double tax_amount = calculate_percentage(after_discount, calc_data.shop_tax_rate);
    double final_total = after_discount + tax_amount;
    ESP_LOGI(TAG, "║ 🏛️ ภาษี %.0f%%:                 +%.2f บาท ║", 
             calc_data.shop_tax_rate, tax_amount);
    ESP_LOGI(TAG, "║ 💳 ยอดชำระสุทธิ:              %.2f บาท ║", final_total);
    ESP_LOGI(TAG, "╚════════════════════════════════════════════╝");
    
    // บันทึกประวัติการขาย
    save_to_history(OP_DISCOUNT, calc_data.shop_total, calc_data.shop_discount, 
                    after_discount, "การขายหน้าร้าน");
}

// 📊 ฟังก์ชันโหมดประวัติ
void history_mode(void) {
    ESP_LOGI(TAG, "\n📊 === โหมดประวัติ ===");
    
    if (calc_data.history_count == 0) {
        ESP_LOGI(TAG, "📝 ยังไม่มีประวัติการคำนวณ");
        return;
    }
    
    ESP_LOGI(TAG, "╔════════════════════════════════════════════════════════╗");
    ESP_LOGI(TAG, "║                    📋 ประวัติการคำนวณ                  ║");
    ESP_LOGI(TAG, "╠════════════════════════════════════════════════════════╣");
    
    // แสดงประวัติล่าสุด 5 รายการ
    int start = calc_data.history_count > 5 ? calc_data.history_count - 5 : 0;
    for (int i = start; i < calc_data.history_count; i++) {
        calculation_history_t* entry = &calc_data.history[i];
        ESP_LOGI(TAG, "║ #%03d │ %s │ %s ║", 
                 entry->id, entry->timestamp, entry->description);
    }
    
    ESP_LOGI(TAG, "╚════════════════════════════════════════════════════════╝");
    
    // สถิติการใช้งาน
    ESP_LOGI(TAG, "\n📈 สถิติการใช้งาน:");
    ESP_LOGI(TAG, "╔═══════════════════════════════════════╗");
    ESP_LOGI(TAG, "║          📊 สรุปการใช้งาน           ║");
    ESP_LOGI(TAG, "╠═══════════════════════════════════════╣");
    ESP_LOGI(TAG, "║ 🔢 การคำนวณทั้งหมด: %d ครั้ง       ║", calc_data.total_calculations);
    ESP_LOGI(TAG, "║ ⏱️ เวลารวม: %.2f มิลลิวินาที       ║", calc_data.total_computation_time);
    
    if (calc_data.total_calculations > 0) {
        double avg_time = calc_data.total_computation_time / calc_data.total_calculations;
        ESP_LOGI(TAG, "║ ⚡ เวลาเฉลี่ย: %.3f มิลลิวินาที     ║", avg_time);
        ESP_LOGI(TAG, "║ 🚀 ประสิทธิภาพ: %s                ║", 
                 avg_time < 1.0 ? "ยอดเยี่ยม" : avg_time < 5.0 ? "ดี" : "ปกติ");
    }
    
    ESP_LOGI(TAG, "║ ⭐ ความแม่นยำ: 100%%               ║");
    ESP_LOGI(TAG, "╚═══════════════════════════════════════╝");
}

// 🎮 ฟังก์ชันจำลองการเลือกเมนู
void simulate_menu_navigation(void) {
    // จำลองการนำทางผ่านเมนูต่างๆ
    int demo_sequence[] = {1, 2, 3, 4}; // Basic, Advanced, Shop, History
    
    for (int i = 0; i < 4; i++) {
        show_main_menu();
        ESP_LOGI(TAG, "🎯 เลือกเมนู: %d", demo_sequence[i]);
        vTaskDelay(pdMS_TO_TICKS(2000));
        
        switch (demo_sequence[i]) {
            case 1:
                basic_calculator_mode();
                break;
            case 2:
                advanced_calculator_mode();
                break;
            case 3:
                shop_mode();
                break;
            case 4:
                history_mode();
                break;
        }
        
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}

// 🏁 ฟังก์ชันสรุปและจบโปรแกรม
void show_final_summary(void) {
    ESP_LOGI(TAG, "\n🎉 === ขอบคุณที่ใช้งาน ===");
    ESP_LOGI(TAG, "╔════════════════════════════════════════════════════╗");
    ESP_LOGI(TAG, "║           🧮 เครื่องคิดเลขครบครัน v%s           ║", VERSION);
    ESP_LOGI(TAG, "╠════════════════════════════════════════════════════╣");
    ESP_LOGI(TAG, "║ ✅ การคำนวณทั้งหมด: %d ครั้ง                     ║", calc_data.total_calculations);
    ESP_LOGI(TAG, "║ ⏱️ เวลาที่ใช้รวม: %.2f มิลลิวินาที                ║", calc_data.total_computation_time);
    ESP_LOGI(TAG, "║ 🏆 ประสิทธิภาพ: เยี่ยม                           ║");
    ESP_LOGI(TAG, "║ 🛡️ ความปลอดภัย: สูงสุด                          ║");
    ESP_LOGI(TAG, "╚════════════════════════════════════════════════════╝");
    
    ESP_LOGI(TAG, "\n🎓 สิ่งที่ได้เรียนรู้:");
    ESP_LOGI(TAG, "✅ การเขียนโปรแกรม ESP32 ด้วย C");
    ESP_LOGI(TAG, "✅ การจัดการข้อผิดพลาดแบบมืออาชีพ");
    ESP_LOGI(TAG, "✅ การสร้างระบบเมนูและ UI");
    ESP_LOGI(TAG, "✅ การคำนวณคณิตศาสตร์ขั้นสูง");
    ESP_LOGI(TAG, "✅ การประยุกต์ใช้ในงานจริง");
    
    ESP_LOGI(TAG, "\n🚀 คุณพร้อมสำหรับโปรเจคถัดไปแล้ว!");
    ESP_LOGI(TAG, "💝 ขอบคุณและขอให้โชคดี!");
}

void app_main(void) {
    ESP_LOGI(TAG, "🚀 เริ่มต้นเครื่องคิดเลขครบครัน!");
    
    // รอให้ระบบเริ่มต้นเสร็จสิ้น
    vTaskDelay(pdMS_TO_TICKS(1000));
    
    // แสดง Logo
    show_logo();
    vTaskDelay(pdMS_TO_TICKS(2000));
    
    // เริ่มต้นข้อมูล
    calc_data.current_mode = MODE_MAIN_MENU;
    calc_data.shop_tax_rate = 7.0;
    
    ESP_LOGI(TAG, "⚡ ระบบพร้อมใช้งาน!");
    ESP_LOGI(TAG, "🛡️ ระบบป้องกันข้อผิดพลาดเปิดใช้งาน");
    ESP_LOGI(TAG, "💾 ระบบบันทึกประวัติพร้อม");
    vTaskDelay(pdMS_TO_TICKS(1500));
    
    // จำลองการใช้งานผ่านเมนูต่างๆ
    simulate_menu_navigation();
    
    // แสดงสรุปท้าย
    show_final_summary();
    
    ESP_LOGI(TAG, "\n🎯 โปรแกรมเสร็จสิ้น - ขอบคุณที่ใช้งาน!");
}
```
