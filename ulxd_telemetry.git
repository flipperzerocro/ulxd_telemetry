#include <furi.h>
#include <gui/gui.h>
#include <input/input.h>
#include <subghz/subghz.h>
#include <stdio.h>

#define FREQUENCY_STEP 100000 // Frequency step: 100 kHz
#define FREQUENCY_MIN 470000000 // Minimum frequency: 470 MHz
#define FREQUENCY_MAX 530000000 // Maximum frequency: 530 MHz

typedef struct {
    uint32_t frequency;
    bool running;
    SubGhz* subghz;
    ViewPort* viewport;
    Gui* gui;
} ULXDTelemetryApp;

static void update_display(ULXDTelemetryApp* app) {
    char buffer[64];

    snprintf(buffer, sizeof(buffer), "Frequency: %.3f MHz", app->frequency / 1000000.0);
    canvas_clear(app->viewport->canvas);
    canvas_set_font(app->viewport->canvas, FontPrimary);
    canvas_draw_str(app->viewport->canvas, 10, 20, buffer);
    canvas_draw_str(app->viewport->canvas, 10, 40, "Receiving telemetry...");
    canvas_draw_str(app->viewport->canvas, 10, 60, "Signal: -XX dBm");
    canvas_draw_str(app->viewport->canvas, 10, 80, "Battery: X%");
}

static void initialize_subghz(ULXDTelemetryApp* app) {
    subghz_stop_rx(app->subghz);
    subghz_set_frequency(app->subghz, app->frequency);
    subghz_set_protocol(app->subghz, SubGhzProtocol2FSK);
    subghz_set_bitrate(app->subghz, 250000); // Example bitrate
    subghz_start_rx(app->subghz);
}

static void handle_input(InputEvent* input_event, void* context) {
    ULXDTelemetryApp* app = context;

    if(input_event->type == InputTypePress || input_event->type == InputTypeRepeat) {
        switch(input_event->key) {
            case InputKeyUp:
                if(app->frequency + FREQUENCY_STEP <= FREQUENCY_MAX) {
                    app->frequency += FREQUENCY_STEP;
                    initialize_subghz(app);
                }
                break;
            case InputKeyDown:
                if(app->frequency - FREQUENCY_STEP >= FREQUENCY_MIN) {
                    app->frequency -= FREQUENCY_STEP;
                    initialize_subghz(app);
                }
                break;
            case InputKeyBack:
                app->running = false;
                break;
            default:
                break;
        }
    }
}

int32_t ulxd_telemetry_main(void* p) {
    UNUSED(p);

    ULXDTelemetryApp app = {
        .frequency = FREQUENCY_MIN,
        .running = true,
        .subghz = subghz_alloc(),
        .viewport = viewport_alloc(),
        .gui = furi_record_open(RECORD_GUI),
    };

    viewport_set_draw_callback(app.viewport, (ViewportDrawCallback)update_display, &app);
    viewport_set_input_callback(app.viewport, handle_input, &app);
    gui_add_viewport(app.gui, app.viewport, GuiLayerFullscreen);

    initialize_subghz(&app);

    while(app.running) {
        furi_delay_ms(100);
        gui_update(app.gui);
    }

    subghz_stop_rx(app.subghz);
    subghz_free(app.subghz);
    gui_remove_viewport(app.gui, app.viewport);
    viewport_free(app.viewport);
    furi_record_close(RECORD_GUI);

    return 0;
}
