# IT-Request-Management-System
ระบบจัดการคำร้องขอด้าน IT ภายในบริษัทถูกพัฒนาขึ้นเพื่อให้การบันทึกข้อมูล การติดตามสถานะ และการอนุมัติคำร้องขอด้าน IT ภายในบริษัทเป็นไปได้อย่างง่ายดาย ข้อมูลจึงถูกต้องและเป็นปัจจุบันเสมอ ผู้ใช้งานหลายคนสามารถทำงานบนข้อมูลเดียวกันได้ในเวลาเดียวกัน


## โปรเจกต์: IT-Request-Management-System
![Project Status](https://img.shields.io/badge/Status-Completed-success) 
![Data Integrity](https://img.shields.io/badge/Data%20Integrity-Middle-yellow)
![Tech Stack](https://img.shields.io/badge/Stack-PowerApps%20%7C%20SharePoint%20%7C%20Data%20Modeling-orange)

##  Executive Summary (สรุปภาพรวมในมุม Data)
 Application นี้ถูกพัฒนาขึ้นเพื่อแก้ปัญหาความล่าช้าและความผิดพลาดจากการจัดการคำร้องแบบเดิม (Manual Process) ระบบนี้ช่วยให้พนักงานสามารถส่งคำร้อง ติดตามสถานะ และได้รับการอนุมัติอย่างรวดเร็ว

จุดเด่นสำคัญคือการรองรับได้หลายคน ทำให้เจ้าหน้าที่ IT และผู้เกี่ยวข้องสามารถทำงานบนฐานข้อมูลเดียวกันได้แบบ Real-time ข้อมูลจึงมีความถูกต้อง (Accuracy) และเป็นปัจจุบันเสมอ (Up-to-date)

---

##  The Business Challenge & Data Gap
ก่อนเริ่มพัฒนาระบบ กระบวนการแจ้งปัญหา IT อาศัยการสื่อสารผ่านหลายช่องทาง (เช่น อีเมล, ไลน์, หรือปากต่อปาก) และการกรอกแบบฟอร์มกระดาษ ซึ่งก่อให้เกิดปัญหาสำคัญ:
* **Lack of Visibility:** ผู้แจ้งไม่ทราบสถานะงานของตนเองว่าดำเนินการถึงขั้นตอนไหน
* **High Turnaround Time:** กระบวนการอนุมัติล่าช้าเนื่องจากต้องรอเอกสารวิ่งตามลำดับขั้น
* **Lost Requests:** คำร้องบางรายการตกหล่นหรือสูญหายระหว่างทาง

##  Data Architecture & Modeling 
> *Note: เนื่องจากข้อมูลจริงเป็นความลับของบริษัท แผนภาพด้านล่างจึงเป็นการจำลองโครงสร้างข้อมูล (Schema Design) ที่ผมออกแบบเพื่อรองรับ Business Logic*

โครงสร้างฐานข้อมูลของระบบ **IT Request Management System** ออกแบบมาให้รองรับความสัมพันธ์แบบ **Relational Database** เพื่อความยืดหยุ่นและการจัดการข้อมูลที่ถูกต้อง (Data Integrity) โดยประกอบด้วย 4 Entity หลัก ดังนี้
```mermaid
erDiagram
    USERS ||--o{ REQUESTS : "creates"
    USERS ||--o{ TASKS : "assigned_to"
    REQUESTS ||--|{ TASKS : "generates"
    REQUESTS ||--o{ APPROVAL_LOGS : "has"

    USERS {
        string email PK "User Email (Unique ID)"
        string name "Full Name"
        string role "Role (User, IT_Staff, Manager, Admin)"
        string factory "Factory Location"
        string department "Department Name"
    }

    REQUESTS {
        string request_id PK "Format: IT-YYMM-XXX"
        string requester_email FK
        string title "Request Title"
        string category "Service Category"
        text description "Problem Detail"
        string ref_image_url "Attachment URL"
        string status_code "Current Status (01-08)"
        string dept_manager_email "Approver 1"
        string div_manager_email "Approver 2"
        string factory_manager_email "Approver 3"
        enum operating_cost "None, <=10k, >10k"
        datetime created_at
        datetime estimated_completion_date
    }

    TASKS {
        string task_id PK
        string request_id FK
        string assigned_to_email FK "IT Staff Email"
        enum priority "Low, Medium, High"
        enum task_status "Not Started, In Progress, Completed"
        date start_date
        date due_date
        text resolution_details "Fix Description"
        string resolution_attachment_url "File/Image URL"
    }

    APPROVAL_LOGS {
        int log_id PK
        string request_id FK
        string approver_email FK
        string action "Approve / Reject / Return"
        string comment "Reason or Note"
        datetime timestamp
    }
```

##  System Workflow (ขั้นตอนการทำงาน)
```mermaid
graph TD
    Start((Start)) --> User["User Submit Request"]
    User -->|Require Approval| S01["01-รอผู้จัดการแผนกอนุมัติ"]
    User -->|General Request| S02["02-รอผู้จัดการแผนก IT อนุมัติ"]
    S01 --> S02
    S02 -- "Estimate Cost" --> CheckCost{"Check Cost"}
    CheckCost -- "Cost <= 10,000" --> S05["05-รอดำเนินการ"]
    CheckCost -- "Cost > 10,000" --> S03["03-รอผู้จัดการฝ่ายอนุมัติ"]
    S03 --> S04["04-รอผู้จัดการโรงงานอนุมัติ"]
    S04 --> S05
    S05 -->|IT Manager Assign Task| S06["06-กำลังดำเนินการ"]
    S06 -->|Staff Fix & IT Mgr Review| S07["07-ดำเนินการเสร็จสิ้น"]
    S07 --> UserCheck{"User Verify"}
    UserCheck -- "Satisfied" --> S08(("08-ปิดคำร้อง"))
    UserCheck -- "Not Satisfied" --> Return["ส่งกลับเพื่อแก้ไข"]
    Return -->|"ระบบแจ้งเตือนกลับไปยังIT Manager"| S05
```
---
# Example UI
<img width="782" height="434" alt="image" src="https://github.com/user-attachments/assets/bf395fdf-3662-4fd9-8aad-f2d80078485e" />


## Conclusion

**IT Request Management System** ถูกพัฒนาขึ้นเพื่อเพิ่มประสิทธิภาพในการจัดการคำร้องขอด้าน IT ภายในองค์กรให้มีความถูกต้องและเป็นปัจจุบันเสมอ  
โดยเปลี่ยนจากการอนุมัติและติดตามงานแบบเดิม มาสู่ระบบ Digital ที่รองรับการทำงานร่วมกันหลายคน (Multi-user) เต็มรูปแบบ

**สิ่งที่ระบบนี้มอบให้:**
* **Centralized Data:** รวบรวมทุกคำร้องขอ (Requests) และประวัติการแก้ไขไว้ในที่เดียว ง่ายต่อการติดตามสถานะและตรวจสอบข้อมูลย้อนหลัง 
* **Streamlined Workflow:** ลดความสับสนในขั้นตอนการอนุมัติและการมอบหมายงาน (Task Assignment) ด้วยลำดับขั้นที่ชัดเจนตั้งแต่การแจ้งเรื่อง, การประเมินค่าใช้จ่าย จนถึงการปิดงาน 
* **Real-time Decision:** ทีม IT และผู้บริหารสามารถเห็นสถานะงานล่าสุดพร้อมกันผ่าน Dashboard ช่วยให้การบริหารจัดการและการตัดสินใจเป็นไปอย่างรวดเร็ว

โปรเจกต์นี้ไม่เพียงแค่ช่วยลดความผิดพลาดของข้อมูล แต่ยังสร้างมาตรฐานการให้บริการ (Service Standard) และความโปร่งใสในการดำเนินงานของฝ่าย IT อย่างเป็นระบบ

