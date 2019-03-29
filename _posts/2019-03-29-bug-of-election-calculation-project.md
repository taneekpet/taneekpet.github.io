---
title: Bug of election calculation project
tags:
  - Election
  - Opensource
date: 2019-03-29 07:40:00 +0700
---

จากที่ได้ทำ[เว็บคำนวณผลคะแนนเลือกตั้ง2019][projects]เล่นๆไปคราวก่อน

วันนี้ กกต. ประกาศคะแนนอย่างเป็นทางการออกมา ซึ่งเป็นรูปแบบไฟล์ pdf
แล้วมีคนแปลงเป็น google sheets ให้[ตามลิงค์นี้][score-google-sheets]

ผมก็เลยเอาไปแปลงในรูปแบบของ csv ที่ตัวเว็บที่ทำรองรับ ได้เป็น[ไฟล์นี้][score-parsed]

ปรากฎว่าเจอข้อผิดพลาดสองประการในเว็บของตัวเอง
- ลืมทำให้ csv รองรับคะแนนไม่ออกเสียง ผลการคำนวณเก้าอี้บัญชีรายชื่อเลยคลาดเคลื่อนไป
(เทียบกับ[ผลการคำนวณของ workpoint][workpoint])
- เวลาที่ใช้ประมวลผลจากไฟล์ csv นานมาก (ประมาณครึ่ง ชม.)
จากการสังเกตพบว่าเกิดจากการเขียนโค้ด HTML ไม่มีประสิทธิภาพ ผมใช้วิธีเพิ่ม DOM Element
สำหรับทุกการเพิ่มพรรค/เขต และการเพิ่มก็ไม่ได้เขียนให้ optimize อะไรเลย
การเพิ่มทั้งหมด 350 เขต * ประมาณ 80 พรรค
การเพิ่ม DOM Element แบบไม่มีประสิทธิภาพจำนวนเกือบ 30,000 ครั้งเลยกินเวลานานมาก
แถมระหว่างนั้นก็ทำให้หน้าเว็บมัน not responding ไปซะอีก

เดี๋ยวไว้จะหาโอกาสปรับปรุงแก้ข้อผิดพลาดทั้งสองอันเสร็จแล้วจะอัพเดทในหน้า [projects][project-page] ครับ

[project]: //thai-election2019/
[project-page]: /projects
[score-google-sheets]: //docs.google.com/spreadsheets/d/1xKLvpNFD9JtIY7HJ8vk1AjlVUAT-AjaQrvKDBDv6Tu4/edit#gid=1145377028
[score-parsed]: /assets/file/parsed-score.csv
[workpoint]: //vote.workpointnews.com