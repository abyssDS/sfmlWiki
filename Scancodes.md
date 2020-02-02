# Links

- [Forum discussion](https://en.sfml-dev.org/forums/index.php?topic=13692.0)
- [Issue Ticket](https://github.com/SFML/SFML/issues/7)
- [Main development branch & PR](https://github.com/SFML/SFML/pull/1235)
- [Elias' Linux branch](https://github.com/eliasdaler/SFML/tree/scancode-linux)
- [Test Example](https://github.com/mantognini/SFML-Test-Events/tree/scancode)

# Current Situation

## Key API

| OS      | Real-Time           | Event                       |
|---------|---------------------|-----------------------------|
| Windows | GetAsyncKeyState    | WM_SYSKEYDOWN / WM_SYSKEYUP |
| Linux   | XKeysymToKeycode    | KeyPress / KeyRelease       |
| macOS   | IOHIDDeviceGetValue |                             |
| Android |                     |                             |

## Key Mapping

| Key       | Enum Code | Windows Mapping | Linux Mapping   | macOS Mapping                  | Android Mapping        |
|-----------|-----------|-----------------|-----------------|--------------------------------|------------------------|
| Unknown   |        -1 |                 |                 |                                |                        |
| A         |         0 | 'A'             | XK_a            |                                | AKEYCODE_A             |
| B         |         1 | 'B'             | XK_b            |                                | AKEYCODE_B             |
| C         |         2 | 'C'             | XK_c            |                                | AKEYCODE_C             |
| D         |         3 | 'D'             | XK_d            |                                | AKEYCODE_D             |
| E         |         4 | 'E'             | XK_e            |                                | AKEYCODE_E             |
| F         |         5 | 'F'             | XK_f            |                                | AKEYCODE_F             |
| G         |         6 | 'G'             | XK_g            |                                | AKEYCODE_G             |
| H         |         7 | 'H'             | XK_h            |                                | AKEYCODE_H             |
| I         |         8 | 'I'             | XK_i            |                                | AKEYCODE_I             |
| J         |         9 | 'J'             | XK_j            |                                | AKEYCODE_J             |
| K         |        10 | 'K'             | XK_k            |                                | AKEYCODE_K             |
| L         |        11 | 'L'             | XK_l            |                                | AKEYCODE_L             |
| M         |        12 | 'M'             | XK_m            |                                | AKEYCODE_M             |
| N         |        13 | 'N'             | XK_n            |                                | AKEYCODE_N             |
| O         |        14 | 'O'             | XK_o            |                                | AKEYCODE_O             |
| P         |        15 | 'P'             | XK_p            |                                | AKEYCODE_P             |
| Q         |        16 | 'Q'             | XK_q            |                                | AKEYCODE_Q             |
| R         |        17 | 'R'             | XK_r            |                                | AKEYCODE_R             |
| S         |        18 | 'S'             | XK_s            |                                | AKEYCODE_S             |
| T         |        19 | 'T'             | XK_t            |                                | AKEYCODE_T             |
| U         |        20 | 'U'             | XK_u            |                                | AKEYCODE_U             |
| V         |        21 | 'V'             | XK_v            |                                | AKEYCODE_V             |
| W         |        22 | 'W'             | XK_w            |                                | AKEYCODE_W             |
| X         |        23 | 'X'             | XK_x            |                                | AKEYCODE_X             |
| Y         |        24 | 'Y'             | XK_y            |                                | AKEYCODE_Y             |
| Z         |        25 | 'Z'             | XK_z            |                                | AKEYCODE_Z             |
| Num0      |        26 | '0'             | XK_0            |                                | AKEYCODE_0             |
| Num1      |        27 | '1'             | XK_1            |                                | AKEYCODE_1             |
| Num2      |        28 | '2'             | XK_2            |                                | AKEYCODE_2             |
| Num3      |        29 | '3'             | XK_3            |                                | AKEYCODE_3             |
| Num4      |        30 | '4'             | XK_4            |                                | AKEYCODE_4             |
| Num5      |        31 | '5'             | XK_5            |                                | AKEYCODE_5             |
| Num6      |        32 | '6'             | XK_6            |                                | AKEYCODE_6             |
| Num7      |        33 | '7'             | XK_7            |                                | AKEYCODE_7             |
| Num8      |        34 | '8'             | XK_8            |                                | AKEYCODE_8             |
| Num9      |        35 | '9'             | XK_9            |                                | AKEYCODE_9             |
| Escape    |        36 | VK_ESCAPE       | XK_Escape       | 0x35                           | AKEYCODE_BACK          |
| LControl  |        37 | VK_CONTROL      | XK_Control_L    | 0x3b                           |                        |
| LShift    |        38 | VK_SHIFT        | XK_Shift_L      | 0x38                           | AKEYCODE_SHIFT_LEFT    |
| LAlt      |        39 | VK_MENU         | XK_Alt_L        | 0x3a                           | AKEYCODE_ALT_LEFT      |
| LSystem   |        40 | VK_LWIN         | XK_Super_L      | 0x37                           |                        |
| RControl  |        41 | VK_CONTROL      | XK_Control_R    | 0x3e                           |                        |
| RShift    |        42 | VK_SHIFT        | XK_Shift_R      | 0x3c                           | AKEYCODE_SHIFT_RIGHT   |
| RAlt      |        43 | VK_MENU         | XK_Alt_R        | 0x3d                           | AKEYCODE_ALT_RIGHT     |
| RSystem   |        44 | VK_RWIN         | XK_Super_R      | 0x36                           |                        |
| Menu      |        45 | VK_APPS         | XK_Menu         | 0x7f / NSMenuFunctionKey       |                        |
| LBracket  |        46 | VK_OEM_4        | XK_bracketleft  | 0x21                           | AKEYCODE_LEFT_BRACKET  |
| RBracket  |        47 | VK_OEM_6        | XK_bracketright | 0x1e                           | AKEYCODE_RIGHT_BRACKET |
| Semicolon |        48 | VK_OEM_1        | XK_semicolon    | 0x29                           | AKEYCODE_SEMICOLON     |
| Comma     |        49 | VK_OEM_COMMA    | XK_comma        | 0x2b                           | AKEYCODE_COMMA         |
| Period    |        50 | VK_OEM_PERIOD   | XK_period       | 0x41 / 0x2f                    | AKEYCODE_PERIOD        |
| Quote     |        51 | VK_OEM_7        | XK_apostrophe   | 0x27                           | AKEYCODE_APOSTROPHE    |
| Slash     |        52 | VK_OEM_2        | XK_slash        | 0x2c                           | AKEYCODE_SLASH         |
| Backslash |        53 | VK_OEM_5        | XK_backslash    | 0x2a                           | AKEYCODE_BACKSLASH     |
| Tilde     |        54 | VK_OEM_3        | XK_grave        | 0x0a                           | AKEYCODE_GRAVE         |
| Equal     |        55 | VK_OEM_PLUS     | XK_equal        | 0x51 / 0x18                    | AKEYCODE_EQUALS        |
| Hyphen    |        56 | VK_OEM_MINUS    | XK_minus        | 0x32                           | AKEYCODE_MINUS         |
| Space     |        57 | VK_SPACE        | XK_space        | 0x31                           | AKEYCODE_SPACE         |
| Enter     |        58 | VK_RETURN       | XK_Return       | 0x4c / 0x24                    | AKEYCODE_ENTER         |
| Backspace |        59 | VK_BACK         | XK_BackSpace    | 0x33                           | AKEYCODE_DEL           |
| Tab       |        60 | VK_TAB          | XK_Tab          | 0x30                           | AKEYCODE_TAB           |
| PageUp    |        61 | VK_PRIOR        | XK_Prior        | 0x74 / NSPageUpFunctionKey     | AKEYCODE_PAGE_UP       |
| PageDown  |        62 | VK_NEXT         | XK_Next         | 0x79 / NSPageDownFunctionKey   | AKEYCODE_PAGE_DOWN     |
| End       |        63 | VK_END          | XK_End          | 0x77 / NSEndFunctionKey        |                        |
| Home      |        64 | VK_HOME         | XK_Home         | 0x73 / NSHomeFunctionKey       |                        |
| Insert    |        65 | VK_INSERT       | XK_Insert       | 0x72 / NSInsertFunctionKey     |                        |
| Delete    |        66 | VK_DELETE       | XK_Delete       | 0x75 / NSDeleteFunctionKey     | AKEYCODE_FORWARD_DEL   |
| Add       |        67 | VK_ADD          | XK_KP_Add       | 0x45                           |                        |
| Subtract  |        68 | VK_SUBTRACT     | XK_KP_Subtract  | 0x4e                           |                        |
| Multiply  |        69 | VK_MULTIPLY     | XK_KP_Multiply  | 0x43                           |                        |
| Divide    |        70 | VK_DIVIDE       | XK_KP_Divide    | 0x4b                           |                        |
| Left      |        71 | VK_LEFT         | XK_Left         | 0x7b / NSLeftArrowFunctionKey  |                        |
| Right     |        72 | VK_RIGHT        | XK_Right        | 0x7c / NSRightArrowFunctionKey |                        |
| Up        |        73 | VK_UP           | XK_Up           | 0x7e / NSUpArrowFunctionKey    |                        |
| Down      |        74 | VK_DOWN         | XK_Down         | 0x7d / NSDownArrowFunctionKey  |                        |
| Numpad0   |        75 | VK_NUMPAD0      | XK_KP_Insert    | 0x52                           |                        |
| Numpad1   |        76 | VK_NUMPAD1      | XK_KP_End       | 0x53                           |                        |
| Numpad2   |        77 | VK_NUMPAD2      | XK_KP_Down      | 0x54                           |                        |
| Numpad3   |        78 | VK_NUMPAD3      | XK_KP_Page_Down | 0x55                           |                        |
| Numpad4   |        79 | VK_NUMPAD4      | XK_KP_Left      | 0x56                           |                        |
| Numpad5   |        80 | VK_NUMPAD5      | XK_KP_Begin     | 0x57                           |                        |
| Numpad6   |        81 | VK_NUMPAD6      | XK_KP_Right     | 0x58                           |                        |
| Numpad7   |        82 | VK_NUMPAD7      | XK_KP_Home      | 0x59                           |                        |
| Numpad8   |        83 | VK_NUMPAD8      | XK_KP_Up        | 0x5b                           |                        |
| Numpad9   |        84 | VK_NUMPAD9      | XK_KP_Page_Up   | 0x5c                           |                        |
| F1        |        85 | VK_F1           | XK_F1           | 0x7a / NSF1FunctionKey         |                        |
| F2        |        86 | VK_F2           | XK_F2           | 0x78 / NSF2FunctionKey         |                        |
| F3        |        87 | VK_F3           | XK_F3           | 0x63 / NSF3FunctionKey         |                        |
| F4        |        88 | VK_F4           | XK_F4           | 0x76 / NSF4FunctionKey         |                        |
| F5        |        89 | VK_F5           | XK_F5           | 0x60 / NSF5FunctionKey         |                        |
| F6        |        90 | VK_F6           | XK_F6           | 0x61 / NSF6FunctionKey         |                        |
| F7        |        91 | VK_F7           | XK_F7           | 0x62 / NSF7FunctionKey         |                        |
| F8        |        92 | VK_F8           | XK_F8           | 0x64 / NSF8FunctionKey         |                        |
| F9        |        93 | VK_F9           | XK_F9           | 0x65 / NSF9FunctionKey         |                        |
| F10       |        94 | VK_F10          | XK_F10          | 0x6d / NSF10FunctionKey        |                        |
| F11       |        95 | VK_F11          | XK_F11          | 0x67 / NSF11FunctionKey        |                        |
| F12       |        96 | VK_F12          | XK_F12          | 0x6f / NSF12FunctionKey        |                        |
| F13       |        97 | VK_F13          | XK_F13          | 0x69 / NSF13FunctionKey        |                        |
| F14       |        98 | VK_F14          | XK_F14          | 0x6b / NSF14FunctionKey        |                        |
| F15       |        99 | VK_F15          | XK_F15          | 0x71 / NSF15FunctionKey        |                        |
| Pause     |       100 | VK_PAUSE        | XK_Pause        | NSPauseFunctionKey             |                        |
| KeyCount  |       101 |                 |                 |                                |                        |
| Dash      |        56 |                 |                 |                                |                        |
| BackSpace |        59 |                 |                 |                                |                        |
| BackSlash |        52 |                 |                 |                                |                        |
| SemiColon |        48 |                 |                 |                                |                        |
| Return    |        58 |                 |                 |                                |                        |