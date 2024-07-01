# esp32-c3-ldckit
Basic configuration for esp32-c3-ldckit espressig

*WARNING*: the code below results in a "bug/problem" whose log is below. Despite this, the code is functional.
        
        [13:58:58][W][component:237]: Component display took a long time for an operation (290 ms).
        [13:58:58][W][component:238]: Components should block for at most 30 ms.

The installation and development process was not documented. When possible, I will document the procedure.

    substitutions:
      name: esphome-web-29e6b0
      friendly_name: esp32-c3-lcdkit
    
    esphome:
      name: ${name}
      friendly_name: ${friendly_name}
      min_version: 2024.6.0
      name_add_mac_suffix: false
      project:
        name: esphome.web
        version: '1.0'
    
    esp32:
      board: esp32-c3-devkitm-1
      framework:
        type: arduino
    /# Link for backgrou LCD image
    image:
      - file: 'images/logotipo_ondas_de_pipa.png'
        id: logo_ondas
        type: RGB565
        resize: 200x200
    # Necessary for print time
    time:
      - platform: homeassistant
        id: esptime
    
    # Enable logging
    logger:
    
    # Enable Home Assistant API
    api:
    
    # Allow Over-The-Air updates
    ota:
    - platform: esphome
    
    # Allow provisioning Wi-Fi via serial
    improv_serial:
    
    # Change WI-FI credentials
    wifi:
      ssid: XXXXX
      password: XXXXX
      # Set up a wifi access point (when wifi connection fails, the esp32 create a "AP".
      ap:
        ssid: "LCDkit"
        password: "lcdkit123"
    
    # In combination with the `ap` this allows the user
    # to provision wifi credentials to the device via WiFi AP.
    captive_portal:
    
    dashboard_import:
      package_import_url: github://esphome/firmware/esphome-web/esp32c3.yaml@main
      import_full_config: true
    
    # Sets up Bluetooth LE (Only on ESP32) to allow the user
    # to provision wifi credentials to the device.
    esp32_improv:
      authorizer: none
    
    # To have a "next url" for improv serial
    web_server:
    
    # Example to read a state of binary sensor (on/ff)
    text_sensor:
        - platform: homeassistant
          id: varanda_201
          entity_id: binary_sensor.xxxx1_contact
          internal: true
    
        - platform: homeassistant
          id: varanda_202
          entity_id: binary_sensor.xxxx2_contact
          internal: true
    # Knob configuration. When knob is press (on press) the display will change for next page    
    binary_sensor:
      - platform: gpio
        pin:
          number: GPIO9
          mode: INPUT_PULLUP
          inverted: true
        name: "SW Knob change page"
        on_press:
          then:
            - display.page.show_next: lcd_display

    # Necessary drive for gc9a01 work 
    external_components:
      - source: github://smkent/esphome@main
        components: [ gc9a01 ]
       
    spi:
      clk_pin: GPIO1
      mosi_pin: GPIO0
    
    display:
      - platform: gc9a01
        id: lcd_display
        cs_pin: GPIO7
        dc_pin: GPIO2
        update_interval: 1.25s
        rotation: 270
        width: 240
        height: 240
        pages: # where you configure the pages.
          - id: page1
            lambda: |-
              it.image(20, 20, id(logo_ondas));
              it.printf(120, 35, id(font_16), id(color_black), TextAlign::CENTER, "Text for sensor 1: %s", id(xxxx1).state.c_str());
              it.printf(120, 50, id(font_16), id(color_black), TextAlign::CENTER, "Text for sensor 2: %s", id(xxxx2).state.c_str());
              it.strftime(120, 210, id(font_16), id(color_black), TextAlign::CENTER, "%H:%M", id(esptime).now());
              it.strftime(120, 195, id(font_16), id(color_black), TextAlign::CENTER, "%c", id(esptime).now());
          - id: page2
            lambda: |-
              it.image(20, 20, id(logo_ondas));
              it.printf(120, 35, id(font_16), id(color_black), TextAlign::CENTER, "Text for sensor 1: %s", id(xxxx1).state.c_str());
              it.strftime(120, 200, id(font_16), id(color_black), TextAlign::CENTER, "%X - %x", id(esptime).now());
          - id: page3
            lambda: |-
              it.image(20, 20, id(logo_ondas));
              it.printf(120, 50, id(font_16), id(color_black), TextAlign::CENTER, "Text for sensor 2: %s", id(xxxx2).state.c_str());
              it.strftime(120, 200, id(font_16), id(color_black), TextAlign::CENTER, "%X - %x", id(esptime).now());
          - id: page4
            lambda: |-
              it.image(20, 20, id(logo_ondas));
              it.strftime(120, 200, id(font_16), id(color_black), TextAlign::CENTER, "%X - %x", id(esptime).now());
    # Set interval for change pages without press the know
    interval:
      - interval: 5s
        then:
          - display.page.show_next: lcd_display
          - component.update: lcd_display
        
    font:
      - file: 'fonts/OpenSans-Medium.ttf'
        id: font_16
        size: 16
    
      - file: 'fonts/OpenSans-Medium.ttf'
        id: font_20
        size: 20
    
    color:
      - id: color_black
        red: 0%
        green: 0%
        blue: 0%
    
      - id: color_purple
        red: 27%
        green: 13%
        blue: 37%
