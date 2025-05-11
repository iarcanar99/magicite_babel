## Magicite Babel (MBB) เป็นโปรแกรมแปลภาษาเกมแบบ Real-time ใช้ OCR ดึงข้อความ, แปลด้วย AI Models (Gemini, GPT, Claude), และแสดงผลบน UI ที่ปรับแต่งได้สูง

* **Main Page**
 https://iarcanar99.github.io/magicite_babel


**ฟีเจอร์หลัก:**

* แปลภาษา Real-time จากหน้าจอเกม
* รองรับ AI Models หลากหลาย
* UI ปรับแต่งได้ (Themes, Presets)
* ระบบจัดการข้อมูลตัวละคร (NPC Manager)
* **Hover Translation:** แปลเมื่อเมาส์ชี้ (อัปเดต 20 เม.ย. 2025)
* **Toggle Switches:** ปุ่มเปิด/ปิดฟีเจอร์บน Control UI (อัปเดต 20 เม.ย. 2025)
* ระบบ Preset อัจฉริยะ และการตั้งค่าที่ครอบคลุม

## Area-Preset System: ภาพรวมและความซับซ้อน

ระบบหัวใจหลักที่จัดการพื้นที่แปล (Area A/B/C) และชุดการตั้งค่า (Preset 1-6) มีความสัมพันธ์ซับซ้อนระหว่างหลายโมดูล

**โครงสร้างข้อมูล:**

* **Area:** พิกัดพื้นที่บนหน้าจอ (start\_x, start\_y, end\_x, end\_y)
* **Preset:** ชุดการตั้งค่า ประกอบด้วย:
    * `areas`: พื้นที่ที่ใช้งาน (เช่น "A+B", "C")
    * `coordinates`: พิกัดของแต่ละ Area (มีการ **copy** เสมอเมื่อโหลด/บันทึก)
    * `role`: บทบาท ("dialog", "lore", "choice", "custom")
    * `name/custom_name`: ชื่อที่แสดง

**การไหลของข้อมูลและการ Sync (สำคัญ):**

* **ศูนย์กลาง:** `MBB.py` ทำหน้าที่เป็น State Owner และ Orchestrator หลัก
* **UI หลัก:** `control_ui.py` (Control Panel), `settings_ui.py` (Settings Window)
* **ฟังก์ชันเสริม:** `hover_translation.py`
* **การ Sync:** ข้อมูล Preset และ Area ถูก Sync ผ่าน `MBB.py` (โดยเฉพาะ `MBB.switch_area` และ `MBB.sync_last_used_preset`) ไปยัง UI และ `HoverTranslator`
* **การสลับ Preset:**
    * **จาก Control UI:** User คลิก -> `control_ui._complete_preset_switch` -> ตรวจสอบ Unsaved Changes -> โหลดพิกัด (Copy) -> เรียก `MBB.switch_area` -> `MBB` อัปเดต State -> เรียก `sync_last_used_preset` -> อัปเดต UI (`control_ui.update_display`) -> แจ้ง `HoverTranslator`
    * **จาก Hover:** เมาส์เข้า Area -> `HoverTranslator` ตรวจสอบ (รวม Cooldown และ Unsaved Changes) -> เรียก `control_ui.select_preset` -> Flow เหมือนกดจาก Control UI
* **การเลือก Area (Custom Preset):** User คลิก A/B/C บน Control UI -> `control_ui` อัปเดต State ภายใน -> ตั้งค่า `has_unsaved_changes` -> เรียก `MBB.switch_area` (แต่คง Preset เดิม)
* **การ Crop พื้นที่:** User Crop -> `MBB.finish_selection` -> `settings.set_translate_area` -> ตั้งค่า `has_unsaved_changes` ใน `control_ui` -> **เรียก `update_button_highlights()` เพื่ออัปเดตสถานะปุ่ม Save** (อัปเดต 25 พ.ค. 2025)
* **การ Sync Toggle Switches (Click/Hover Translate):** การกด Toggle บน UI ใดๆ จะเรียกเมธอดใน `MBB.py` ซึ่งจะ:
    1.  อัปเดตค่าใน `settings.py`
    2.  เปิด/ปิดการทำงานของฟังก์ชัน (เช่น `translation_event`, `hover_translator.toggle()`)
    3.  สั่ง **อัปเดตสถานะ Toggle บน UI ทั้งสอง** (`control_ui` และ `settings_ui`) เพื่อให้แสดงผลตรงกัน

**การแสดงพื้นที่แปล:**

* `MBB.py` จัดการการแสดงกรอบถาวร (`_refresh_area_overlay`) และชั่วคราว (`_show_animated_area`, `_show_quick_area`)

**การสลับพื้นที่อัตโนมัติ:**

* ใช้ `smart_switch_area` และระบบตรวจจับ Dialogue Type ใน `MBB.py`

**พอยน์เตอร์สำคัญ:**

* `MBB.switch_area`: หัวใจการสลับ Area/Preset
* `MBB.sync_last_used_preset`: ศูนย์กลางการ Sync ข้อมูล Preset/Area ข้ามระบบ
* `control_ui._complete_preset_switch`: ตรรกะการสลับ Preset จาก UI
* `control_ui.check_coordinate_changes`: ตรวจสอบการเปลี่ยนแปลงพิกัด (สำคัญสำหรับ Unsaved Changes)
* `control_ui.update_button_highlights`: แสดงสถานะ Unsaved Changes

## Area-Preset System: การแก้ไขและการปรับปรุง (พฤษภาคม 2025)

การปรับปรุงเพื่อแก้ปัญหาการบันทึกและการตรวจสอบการเปลี่ยนแปลง:

1.  **แก้ไขการตรวจสอบ Unsaved Changes (`check_coordinate_changes` ใน `control_ui.py`):**
    * **ปัญหา:** ตรวจสอบไม่ถูกต้องระหว่าง System Preset (1-3) และ Custom Preset (4-6)
    * **แก้ไข:** แยกตรรกะการตรวจสอบ:
        * System Preset: ตรวจสอบเฉพาะการเปลี่ยนแปลง *พิกัด*
        * Custom Preset: ตรวจสอบทั้งการเปลี่ยนแปลง *Area* และ *พิกัด*
2.  **แก้ไขค่าเริ่มต้นไม่สอดคล้อง (`update_button_highlights` ใน `control_ui.py`):**
    * **ปัญหา:** ค่า Default ของ Area String ไม่ตรงกัน ทำให้ตรวจจับ Unsaved Changes ผิดพลาด
    * **แก้ไข:** ใช้ค่า Default ที่สอดคล้องกัน ("" สำหรับ Area ว่าง)
3.  **ปรับปรุงการป้องกัน Race Condition (`process_hover_area` ใน `hover_translation.py`):**
    * **ปัญหา:** Hover อาจสลับ Preset ขณะผู้ใช้กำลังแก้ไข Preset อื่น ทำให้ข้อมูลไม่ถูกบันทึก
    * **แก้ไข:** เพิ่ม **Cooldown** และ **ตรวจสอบ `has_unsaved_changes`** ก่อนทำการสลับ Preset จาก Hover
4.  **ปรับปรุงการจัดการสำเนาข้อมูล (`save_preset` ใน `settings.py`):**
    * **ปัญหา:** การเปลี่ยนแปลงพิกัดใน Preset หนึ่งอาจกระทบ Preset อื่น (Shallow Copy)
    * **แก้ไข:** เน้นการ **ทำสำเนาข้อมูลพิกัดแบบ Deep Copy** เสมอเมื่อบันทึก Preset
5.  **ปรับปรุง Logging:** เพิ่ม Debug Log ที่ชัดเจนขึ้นในส่วนการตรวจสอบ Unsaved Changes
6.  **แก้ไขการอัปเดตปุ่ม Save หลังเลือกพื้นที่ (`MBB.finish_selection`): (อัปเดต 25 พ.ค. 2025)**
    * **ปัญหา:** ปุ่ม Save ไม่ไฮไลท์หลังเลือกพื้นที่ใหม่ใน System Preset (1-3) แม้ว่า `has_unsaved_changes` จะถูกตั้งเป็น `True` แล้ว
    * **สาเหตุ:** โค้ดใน `MBB.finish_selection` พยายามใช้ `itemconfig` กับปุ่ม `tk.Button` ซึ่งไม่รองรับเมธอดนี้ (โครงสร้าง UI เปลี่ยนจาก Canvas เป็น tk.Button)
    * **แก้ไข:** เปลี่ยนจากการเรียกใช้ `itemconfig` เป็นการเรียกใช้ `control_ui.update_button_highlights()` โดยตรง ซึ่งจะจัดการอัปเดตสถานะปุ่ม Save ตามค่า `has_unsaved_changes`

**แนวทางการพัฒนาต่อไป:**

* พิจารณาแยกการจัดการ Preset ตามประเภท (System vs Custom)
* เพิ่มการตรวจสอบการบันทึกสำเร็จและแจ้งผู้ใช้
* พิจารณาระบบ Transaction สำหรับการบันทึกการเปลี่ยนแปลง
* ปรับปรุงให้ UI Event Flow ชัดเจนยิ่งขึ้น โดยเฉพาะความสัมพันธ์ระหว่าง MBB และ UI Components

## โครงสร้างโมดูล

### Core Modules

* **`MBB.py` (Class: `MagicBabelApp`)**
    * **ศูนย์กลางโปรแกรม:** ควบคุม Lifecycle, สร้างและเชื่อมโยง Components
    * **State Owner:** จัดการ State หลัก (`current_area`, `current_preset`, `is_translating`, etc.)
    * **Core Logic:** ควบคุม Loop การแปล, จัดการ Threads
    * **Preset/Area Management:** `switch_area`, `sync_last_used_preset`, `smart_switch_area`, `finish_selection`
    * **Interaction:** `force_translate`, `set_click_translate_mode`, `toggle_hover_translation`
    * **Callback Hub:** เชื่อมต่อระบบย่อย, จัดการแสดง Area ชั่วคราว
    * **UI Synchronization:** ใช้เมธอด `control_ui.update_button_highlights()` เพื่ออัปเดตสถานะ UI (อัปเดต 25 พ.ค. 2025)
* **`settings.py` (Classes: `Settings`, `SettingsUI`)**
    * **`Settings` Class:**
        * จัดการ Config ทั้งหมด (โหลด/บันทึก `settings.json`)
        * Data Access (`get`/`set`)
        * Preset Management (จัดการข้อมูล 6 Presets, **เน้นการ Copy ข้อมูลพิกัดเสมอ**, `find_preset_by_areas`, `get_preset_data`, `save_preset` (ปรับปรุงแล้ว))
    * **`SettingsUI` Class:**
        * หน้าต่างตั้งค่าหลัก
        * จัดการ Callbacks สำหรับ Toggle Switches (ผ่าน `trace_add` เรียกเมธอดใน `MBB.py`)
* **`appearance.py` (Class: `AppearanceManager`)**
    * จัดการ Themes (เริ่มต้น, Custom)
    * แจ้ง Callback เมื่อ Theme เปลี่ยน
    * จัดการ Font และ Style มาตรฐาน

### UI Modules

* **`control_ui.py` (Class: `Control_UI`)**
    * **Control Panel UI:** หน้าต่างควบคุมย่อย
    * **Interaction:** รับ Input (Area, Preset, Save, Force, Toggles)
    * **State Display:** แสดง Area, Preset, สถานะ Toggles, จัดการ Unsaved Changes และปุ่ม Save
    * **Preset System:** จัดการ Layout ปุ่ม Preset, แสดงชื่อ Preset, `check_coordinate_changes` (ปรับปรุง), `update_button_highlights` (ปรับปรุง), `save_preset`
    * **Toggle Handling:** `handle_toggle_change` (ส่ง Callback ไป MBB), `update_..._toggle` (รับคำสั่งจาก MBB เพื่ออัปเดต UI)
    * **Hover-to-Force:** ทำงานเมื่อ Click Translate เปิดอยู่
    * **Button Highlights:** อัปเดตสถานะปุ่ม Save เมื่อมีการเปลี่ยนแปลงผ่าน `update_button_highlights()` (ปรับปรุง 25 พ.ค. 2025)
* **`translated_ui.py` (Class: `Translated_UI`, `ColorAlphaPickerWindow`)**
    * **Translation Display (TUI):** แสดงข้อความแปลบน Canvas
    * **Background Customization:** จัดการสีและความโปร่งใสพื้นหลัง (ย้ายมาจาก `settings_ui`) ผ่าน `ColorAlphaPickerWindow`
    * **Lock Mode:** จัดการ 3 สถานะ (ปลดล็อก, ซ่อนพื้นหลัง, ล็อกตำแหน่ง)
    * **Scrolling, Font, Fadeout Toggle, Feedback Messages**
* **`mini_ui.py` (Class: `MiniUI`)**
    * UI ขนาดย่อ, ปุ่มควบคุมพื้นฐาน, แสดงสถานะ

### Translator Modules

* **`translator_factory.py`:** เลือกและสร้าง Instance ของ Translator
* **`translator_*.py` (Gemini, GPT, Claude):** จัดการ API, สร้าง Prompt, ประมวลผล Response

### Data & Logic Modules

* **`model.py` (Class: `ModelSettings`)**
    * UI ตั้งค่า Model API (เลือก Model, ปรับพารามิเตอร์)
* **`text_corrector.py`:** ประมวลผล OCR ดิบ, ทำความสะอาด, แยกชื่อ/บทสนทนา (ใช้ `EnhancedNameDetector`)
* **`enhanced_name_detector.py`:** อัลกอริทึมช่วยแยกชื่อตัวละคร
* **`npc_manager_card.py`:** ระบบจัดการข้อมูล NPC พร้อม UI
* **`swap_data.py`:** Script ภายนอก (PyQt5) สำหรับสลับ `npc.json`
* **`Manager.py`:** Library จัดการไฟล์ (ใช้โดย `swap_data.py`)
* **`hover_translation.py` (Class: `HoverTranslator`)**
    * **หน้าที่:** ตรวจจับเมาส์ Hover เหนือ **Core Presets (1-3)** เท่านั้น
    * **State Tracking:** ป้องกัน Log Flood (`_currently_hovered_preset`)
    * **Hover Cooldown & Unsaved Check:** ป้องกัน Race Condition (ปรับปรุง)
    * **Interaction:** เรียก `control_ui.select_preset()` โดยตรง
    * **Data Update:** มี `force_update_preset_data()` ให้ MBB เรียก
    * **Callbacks:** ต้องการแค่ `get_screen_scale`, `is_manual_show_area_active`

### Utility Modules

* **`logging_manager.py`:** จัดการ Log ไฟล์, แสดงสถานะ (เพิ่ม Debug Log สำหรับ Preset)
* **`font_manager.py` (Classes: `FontManager`, `FontUI`)**
    * จัดการ Font ระบบ, UI ตั้งค่า Font
* **`screen_capture.py`:** จัดการการจับภาพหน้าจอ
* **`dialogue_cache.py`:** (อาจมี) Cache บริบทการแปล

## Key Workflows / Interactions (สรุป)

* **Theme Change:** `AppearanceManager` -> Callback `MBB` -> `MBB` อัปเดต UI ตัวเองและเรียก `update_theme` ของ Children.
* **Preset Switching (Control UI):** คลิก -> `Control_UI` ตรวจสอบ Unsaved -> โหลดพิกัด (Copy) -> `MBB.switch_area` -> `MBB.sync_last_used_preset` -> อัปเดต `Control_UI` & แจ้ง `HoverTranslator`.
* **Preset Switching (Hover):** เมาส์เข้า Area -> `HoverTranslator` ตรวจสอบ Cooldown/Unsaved -> เรียก `Control_UI.select_preset` -> Flow เหมือน Control UI.
* **Area Selection (Custom Preset):** คลิก A/B/C -> `Control_UI` อัปเดต State -> ตั้ง `has_unsaved_changes` -> เรียก `MBB.switch_area` (คง Preset เดิม).
* **Area Crop:** Crop UI -> `MBB.finish_selection` -> `settings.set_translate_area` -> ตั้งค่า `has_unsaved_changes` ใน `control_ui` -> **เรียก `update_button_highlights()` เพื่ออัปเดตปุ่ม Save** (อัปเดต 25 พ.ค. 2025).
* **Save Preset:** คลิก Save -> `Control_UI.save_preset` -> ทำ Deep Copy พิกัด -> `Settings.save_preset` -> Reset `has_unsaved_changes` -> เรียก `update_button_highlights()` -> แจ้ง `HoverTranslator`.
* **Force Translate:** คลิก Force -> `Control_UI` เรียก `MBB.force_translate`.
* **Auto Area Switching:** `MBB.translation_loop` ตรวจจับ -> `MBB.smart_switch_area` -> `MBB.switch_area`.
* **Toggle Sync:** กด Toggle (UI ใดๆ) -> เรียกเมธอด `MBB` -> `MBB` อัปเดต Setting, เปิด/ปิดฟังก์ชัน, **สั่งอัปเดต UI ทั้งสอง**.
* **Hover-to-Force:** เมาส์ชี้ปุ่ม Force (ถ้า Click Translate เปิด) -> `Control_UI` เรียก `MBB.force_translate`.
