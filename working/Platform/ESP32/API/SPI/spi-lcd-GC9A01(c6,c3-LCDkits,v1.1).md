### GC9A01
#### 介绍
1.28英寸液晶显示模块，IPS屏，65K RGB，240×240分辨率，SPI接口。
#### 规格
- 工作电压：3.3V/5V
- 接口：SPI
- 液晶屏类型：IPS
- 控制器：GC9A01
- 分辨率：240（水平）RGB x 240（垂直）
- 显示尺寸：Φ32.4mm
- 像素尺寸：0.135（H）x 0.135（V）mm
- 尺寸：40.4×37.5(mm) Φ37.5(mm)
* CS:	芯片选择，低电平有效
* 直流:	数据/命令选择（高电平表示数据，低电平表示命令）
* 快速恢复时间:	复位，低电平有效
* BL	背光
### 代码编写
#### 将lcd官方例程移植到esp32-c3-LCDkit
```markdown
#define EXAMPLE_LCD_PIXEL_CLOCK_HZ     BSP_LCD_PIXEL_CLOCK_HZ
#define EXAMPLE_LCD_BK_LIGHT_ON_LEVEL  1
#define EXAMPLE_LCD_BK_LIGHT_OFF_LEVEL !EXAMPLE_LCD_BK_LIGHT_ON_LEVEL
#define EXAMPLE_PIN_NUM_SCLK           BSP_LCD_PCLK
#define EXAMPLE_PIN_NUM_MOSI           BSP_LCD_DATA0
#define EXAMPLE_PIN_NUM_MISO           -1 // There is no MISO pin defined in the esp32_c3_lcdkit.h file
#define EXAMPLE_PIN_NUM_LCD_DC         BSP_LCD_DC
#define EXAMPLE_PIN_NUM_LCD_RST        BSP_LCD_RST
#define EXAMPLE_PIN_NUM_LCD_CS         BSP_LCD_CS
#define EXAMPLE_PIN_NUM_BK_LIGHT       BSP_LCD_BACKLIGHT
```
#### 代码思路
```markdown
1. 包含必要的头文件（行1-15）：这些头文件包含了FreeRTOS任务管理、ESP定时器、LCD面板操作、GPIO驱动、SPI主设备驱动、错误处理、日志记录等功能。
    
2. 定义了一些LCD和SPI的配置参数（行20-43）：这些参数包括SPI主机、像素时钟频率、背光开关电平、SPI引脚编号、LCD分辨率、命令和参数的位数等。
    
3. 定义了一些函数（行50-98）：这些函数主要用于LVGL图形库的刷新和旋转操作。
    
4. `app_main`函数（行104-208）：这是程序的主入口点，它执行以下操作：
    
    - 初始化SPI总线（行117-126）
    - 安装面板IO（行128-142）
    - 安装GC9A01面板驱动（行144-153）
    - 初始化面板并设置面板的颜色反转和镜像（行155-161）
    - 打开面板显示并打开背光（行163-166）
    - 初始化LVGL库并分配绘图缓冲区（行168-177）
    - 注册显示驱动到LVGL（行179-187）
    - 安装LVGL tick定时器（行189-197）
    - 显示LVGL Meter Widget（行200）
    - 在无限循环中处理LVGL的任务和定时器（行202-207）
5. 总的来说，这段代码的主要目标是初始化和配置LCD面板，然后使用LVGL图形库在面板上显示UI。
```

#### spi_lcd_touch_example_main.c
```c++
#include <stdio.h>                // Include the standard input/output library
#include "freertos/FreeRTOS.h"    // Include the FreeRTOS library
#include "freertos/task.h"        // Include the FreeRTOS task management library
#include "esp_timer.h"            // Incl
ude the ESP timer library
#include "esp_lcd_panel_io.h"     // Include the ESP LCD panel IO library
#include "esp_lcd_panel_vendor.h" // Include the ESP LCD panel vendor library
#include "esp_lcd_panel_ops.h"    // Include the ESP LCD panel operations library
#include "driver/gpio.h"          // Include the ESP GPIO driver library
#include "driver/spi_master.h"    // Include the ESP SPI master driver library
#include "esp_err.h"              // Include the ESP error library
#include "esp_log.h"              // Include the ESP log library
#include "lvgl.h"                 // Include the LittlevGL (LVGL) graphics library

#include "esp_lcd_gc9a01.h" // Include the ESP LCD GC9A01 driver library

static const char *TAG = "example"; // Define a tag for logging

// Using SPI2 in the example
#define LCD_HOST SPI2_HOST // Define the SPI host

#define EXAMPLE_LCD_PIXEL_CLOCK_HZ (80 * 1000 * 1000)                 // Define the LCD pixel clock frequency
#define EXAMPLE_LCD_BK_LIGHT_ON_LEVEL 1                               // Define the LCD backlight on level
#define EXAMPLE_LCD_BK_LIGHT_OFF_LEVEL !EXAMPLE_LCD_BK_LIGHT_ON_LEVEL // Define the LCD backlight off level
#define EXAMPLE_PIN_NUM_SCLK 1                                        // Define the SPI clock pin number
#define EXAMPLE_PIN_NUM_MOSI 0                                        // Define the SPI MOSI pin number
#define EXAMPLE_PIN_NUM_MISO -1                                       // Define the SPI MISO pin number
#define EXAMPLE_PIN_NUM_LCD_DC 2                                      // Define the LCD DC pin number
#define EXAMPLE_PIN_NUM_LCD_RST GPIO_NUM_NC                           // Define the LCD reset pin number
#define EXAMPLE_PIN_NUM_LCD_CS 7                                      // Define the LCD CS pin number
#define EXAMPLE_PIN_NUM_BK_LIGHT 5                                    // Define the LCD backlight pin number

// The pixel number in horizontal and vertical
#define EXAMPLE_LCD_H_RES 240 // Define the LCD horizontal resolution
#define EXAMPLE_LCD_V_RES 240 // Define the LCD vertical resolution

// Bit number used to represent command and parameter
#define EXAMPLE_LCD_CMD_BITS 8   // Define the LCD command bits
#define EXAMPLE_LCD_PARAM_BITS 8 // Define the LCD parameter bits

#define EXAMPLE_LVGL_TICK_PERIOD_MS 2 // Define the LVGL tick period in milliseconds

extern void example_lvgl_demo_ui(lv_disp_t *disp); // Declare the function to display the LVGL demo UI

static bool example_notify_lvgl_flush_ready(esp_lcd_panel_io_handle_t panel_io, esp_lcd_panel_io_event_data_t *edata, void *user_ctx)
{
    lv_disp_drv_t *disp_driver = (lv_disp_drv_t *)user_ctx; // Get the LVGL display driver from the user context
    lv_disp_flush_ready(disp_driver);                       // Notify LVGL that the display is ready to be flushed
    return false;                                           // Return false to indicate that the event has been handled
}

static void example_lvgl_flush_cb(lv_disp_drv_t *drv, const lv_area_t *area, lv_color_t *color_map)
{
    esp_lcd_panel_handle_t panel_handle = (esp_lcd_panel_handle_t)drv->user_data; // Get the LCD panel handle from the user data
    int offsetx1 = area->x1;                                                      // Get the start x-coordinate of the area to be flushed
    int offsetx2 = area->x2;                                                      // Get the end x-coordinate of the area to be flushed
    int offsety1 = area->y1;                                                      // Get the start y-coordinate of the area to be flushed
    int offsety2 = area->y2;                                                      // Get the end y-coordinate of the area to be flushed
    // copy a buffer's content to a specific area of the display
    esp_lcd_panel_draw_bitmap(panel_handle, offsetx1, offsety1, offsetx2 + 1, offsety2 + 1, color_map); // Draw the bitmap on the LCD panel
}

/* Rotate display and touch, when rotated screen in LVGL. Called when driver parameters are updated. */
static void example_lvgl_port_update_callback(lv_disp_drv_t *drv)
{
    esp_lcd_panel_handle_t panel_handle = (esp_lcd_panel_handle_t)drv->user_data; // Get the LCD panel handle from the user data

    switch (drv->rotated)
    {                      // Switch on the rotation of the display
    case LV_DISP_ROT_NONE: // Case when the display is not rotated
        // Rotate LCD display
        esp_lcd_panel_swap_xy(panel_handle, false);      // Do not swap x and y
        esp_lcd_panel_mirror(panel_handle, true, false); // Mirror the display horizontally but not vertically
        break;
    case LV_DISP_ROT_90: // Case when the display is rotated 90 degrees
        // Rotate LCD display
        esp_lcd_panel_swap_xy(panel_handle, true);      // Swap x and y
        esp_lcd_panel_mirror(panel_handle, true, true); // Mirror the display both horizontally and vertically
        break;
    case LV_DISP_ROT_180: // Case when the display is rotated 180 degrees
        // Rotate LCD display
        esp_lcd_panel_swap_xy(panel_handle, false);      // Do not swap x and y
        esp_lcd_panel_mirror(panel_handle, false, true); // Mirror the display vertically but not horizontally
        break;
    case LV_DISP_ROT_270: // Case when the display is rotated 270 degrees
        // Rotate LCD display
        esp_lcd_panel_swap_xy(panel_handle, true);        // Swap x and y
        esp_lcd_panel_mirror(panel_handle, false, false); // Do not mirror the display
        break;
    }
}

static void example_increase_lvgl_tick(void *arg)
{
    /* Tell LVGL how many milliseconds has elapsed */
    lv_tick_inc(EXAMPLE_LVGL_TICK_PERIOD_MS); // Increase the LVGL tick by the tick period
}

void app_main(void)
{
    printf("hello world\n");            // Print "hello world" to the console
    static lv_disp_draw_buf_t disp_buf; // contains internal graphic buffer(s) called draw buffer(s)
    static lv_disp_drv_t disp_drv;      // contains callback functions

    ESP_LOGI(TAG, "Turn off LCD backlight"); // Log that the LCD backlight is being turned off
    gpio_config_t bk_gpio_config = {
        // Define the GPIO configuration for the backlight
        .mode = GPIO_MODE_OUTPUT,                        // Set the mode to output
        .pin_bit_mask = 1ULL << EXAMPLE_PIN_NUM_BK_LIGHT // Set the pin bit mask to the backlight pin number
    };
    ESP_ERROR_CHECK(gpio_config(&bk_gpio_config)); // Configure the GPIO for the backlight

    ESP_LOGI(TAG, "Initialize SPI bus"); // Log that the SPI bus is being initialized
    spi_bus_config_t buscfg = {
        // Define the SPI bus configuration
        .sclk_io_num = EXAMPLE_PIN_NUM_SCLK,                          // Set the clock pin number
        .mosi_io_num = EXAMPLE_PIN_NUM_MOSI,                          // Set the MOSI pin number
        .miso_io_num = EXAMPLE_PIN_NUM_MISO,                          // Set the MISO pin number
        .quadwp_io_num = -1,                                          // Set the quad write protect pin number to -1 (not used)
        .quadhd_io_num = -1,                                          // Set the quad hold pin number to -1 (not used)
        .max_transfer_sz = EXAMPLE_LCD_H_RES * 80 * sizeof(uint16_t), // Set the maximum transfer size
    };
    ESP_ERROR_CHECK(spi_bus_initialize(LCD_HOST, &buscfg, SPI_DMA_CH_AUTO)); // Initialize the SPI bus

    ESP_LOGI(TAG, "Install panel IO");          // Log that the panel IO is being installed
    esp_lcd_panel_io_handle_t io_handle = NULL; // Declare a handle for the panel IO
    esp_lcd_panel_io_spi_config_t io_config = {
        // Define the panel IO configuration
        .dc_gpio_num = EXAMPLE_PIN_NUM_LCD_DC,                  // Set the DC pin number
        .cs_gpio_num = EXAMPLE_PIN_NUM_LCD_CS,                  // Set the CS pin number
        .pclk_hz = EXAMPLE_LCD_PIXEL_CLOCK_HZ,                  // Set the pixel clock frequency
        .lcd_cmd_bits = EXAMPLE_LCD_CMD_BITS,                   // Set the LCD command bits
        .lcd_param_bits = EXAMPLE_LCD_PARAM_BITS,               // Set the LCD parameter bits
        .spi_mode = 0,                                          // Set the SPI mode to 0
        .trans_queue_depth = 10,                                // Set the transaction queue depth to 10
        .on_color_trans_done = example_notify_lvgl_flush_ready, // Set the callback function for when a color transaction is done
        .user_ctx = &disp_drv,                                  // Set the user context to the display driver
    };
    // Attach the LCD to the SPI bus
    ESP_ERROR_CHECK(esp_lcd_new_panel_io_spi((esp_lcd_spi_bus_handle_t)LCD_HOST, &io_config, &io_handle)); // Create a new panel IO for the SPI bus

    esp_lcd_panel_handle_t panel_handle = NULL; // Declare a handle for the LCD panel
    esp_lcd_panel_dev_config_t panel_config = {
        // Define the LCD panel configuration
        .reset_gpio_num = EXAMPLE_PIN_NUM_LCD_RST, // Set the reset pin number
        .rgb_endian = LCD_RGB_ENDIAN_BGR,          // Set the RGB endianness to BGR
        .bits_per_pixel = 16,                      // Set the bits per pixel to 16
    };

    ESP_LOGI(TAG, "Install GC9A01 panel driver");                                       // Log that the GC9A01 panel driver is being installed
    ESP_ERROR_CHECK(esp_lcd_new_panel_gc9a01(io_handle, &panel_config, &panel_handle)); // Create a new GC9A01 panel

    ESP_ERROR_CHECK(esp_lcd_panel_reset(panel_handle)); // Reset the LCD panel
    ESP_ERROR_CHECK(esp_lcd_panel_init(panel_handle));  // Initialize the LCD panel

    ESP_ERROR_CHECK(esp_lcd_panel_invert_color(panel_handle, true)); // Invert the color of the LCD panel

    ESP_ERROR_CHECK(esp_lcd_panel_mirror(panel_handle, true, false)); // Mirror the LCD panel horizontally but not vertically

    // user can flush pre-defined pattern to the screen before we turn on the screen or backlight
    ESP_ERROR_CHECK(esp_lcd_panel_disp_on_off(panel_handle, true)); // Turn on the LCD panel display

    ESP_LOGI(TAG, "Turn on LCD backlight");                                  // Log that the LCD backlight is being turned on
    gpio_set_level(EXAMPLE_PIN_NUM_BK_LIGHT, EXAMPLE_LCD_BK_LIGHT_ON_LEVEL); // Set the GPIO level for the backlight to the on level

    ESP_LOGI(TAG, "Initialize LVGL library"); // Log that the LVGL library is being initialized
    lv_init();                                // Initialize the LVGL library
    // alloc draw buffers used by LVGL
    // it's recommended to choose the size of the draw buffer(s) to be at least 1/10 screen sized
    lv_color_t *buf1 = heap_caps_malloc(EXAMPLE_LCD_H_RES * 20 * sizeof(lv_color_t), MALLOC_CAP_DMA); // Allocate the first draw buffer
    assert(buf1);                                                                                     // Assert that the first draw buffer was allocated successfully
    lv_color_t *buf2 = heap_caps_malloc(EXAMPLE_LCD_H_RES * 20 * sizeof(lv_color_t), MALLOC_CAP_DMA); // Allocate the second draw buffer
    assert(buf2);                                                                                     // Assert that the second draw buffer was allocated successfully
    // initialize LVGL draw buffers
    lv_disp_draw_buf_init(&disp_buf, buf1, buf2, EXAMPLE_LCD_H_RES * 20); // Initialize the LVGL draw buffers

    ESP_LOGI(TAG, "Register display driver to LVGL");           // Log that the display driver is being registered to LVGL
    lv_disp_drv_init(&disp_drv);                                // Initialize the display driver
    disp_drv.hor_res = EXAMPLE_LCD_H_RES;                       // Set the horizontal resolution of the display driver
    disp_drv.ver_res = EXAMPLE_LCD_V_RES;                       // Set the vertical resolution of the display driver
    disp_drv.flush_cb = example_lvgl_flush_cb;                  // Set the flush callback of the display driver
    disp_drv.drv_update_cb = example_lvgl_port_update_callback; // Set the update callback of the display driver
    disp_drv.draw_buf = &disp_buf;                              // Set the draw buffer of the display driver
    disp_drv.user_data = panel_handle;                          // Set the user data of the display driver to the LCD panel handle
    lv_disp_t *disp = lv_disp_drv_register(&disp_drv);          // Register the display driver to LVGL

    ESP_LOGI(TAG, "Install LVGL tick timer"); // Log that the LVGL tick timer is being installed
    // Tick interface for LVGL (using esp_timer to generate 2ms periodic event)
    const esp_timer_create_args_t lvgl_tick_timer_args = {
        // Define the LVGL tick timer arguments
        .callback = &example_increase_lvgl_tick, // Set the callback function for the timer
        .name = "lvgl_tick"                      // Set the name of the timer
    };
    esp_timer_handle_t lvgl_tick_timer = NULL;                                                      // Declare a handle for the LVGL tick timer
    ESP_ERROR_CHECK(esp_timer_create(&lvgl_tick_timer_args, &lvgl_tick_timer));                     // Create the LVGL tick timer
    ESP_ERROR_CHECK(esp_timer_start_periodic(lvgl_tick_timer, EXAMPLE_LVGL_TICK_PERIOD_MS * 1000)); // Start the LVGL tick timer

    ESP_LOGI(TAG, "Display LVGL Meter Widget"); // Log that the LVGL Meter Widget is being displayed
    example_lvgl_demo_ui(disp);                 // Display the LVGL demo UI

    while (1)
    { // Start an infinite loop
        // raise the task priority of LVGL and/or reduce the handler period can improve the performance
        vTaskDelay(pdMS_TO_TICKS(10)); // Delay the task for 10 milliseconds
        // The task running lv_timer_handler should have lower priority than that running `lv_tick_inc`
        lv_timer_handler(); // Handle the LVGL timer
    }
}
```

