# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** ปฏิพล พันธุศิลป์

**Student ID / รหัสนิสิต:** 6772055221

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 510 | 595 | 93.5% | 100% |
| t0-small | 14 | 32 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> Large model can buy 21 more accuracy, with the cost of time.

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 29.0 | 227 | 421 | 19 | 0 | 93.9% |
| t1-after | 9.4 | 387 | 1007 | 0 | 25 | 92.6% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> It's misleading because the time is faster than the baseline, but the error column can tell the real story.

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> มันคือทุกคนที่รออยู่ เพราะมันต้องรอ 20 วิ แทนที่จะช้าไปแค่ 10% จาก 200ms เพราะว่า request มันถูกยึด connection ในการเชื่อมกับ large model ที่มี 4 ช่องเอง

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> ลูกค้าไม่ต้องรอก็ได้ ค่อยมาทำทีหลังก็ได้ รอทำ batch

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> If 0.1 it will always timeout with the large model. In the case of 10s that will break the QR2.

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 4 | 539 | 91.5% | 38% |
| t2-0.7 | 0.7 | 362 | 582 | 93.0% | 53% |
| t2-0.8 | 0.8 | 446 | 571 | 93.5% | 64% |
| t2-0.9 | 0.9 | 473 | 592 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> ผมจะเลือก 0.7 เพราะว่ามันยังแม่นยำในระดับที่เกิน 90% และไม่เสี่ยงเกินไป โดยแลกกับ accuracy ไม่ได้สูงที่สุด แต่ก็ไม่ได้ใช้เวลานานที่สุด

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> ฝั่ง p50 ลดเหลือ 362 มันเป็นเพราะ large model มีคิวน้อยลง ส่วน p95 ถูกตัว large ตอบ โดยใช้เวลามาก และการ cascade ทำให้ large ช้ากว่าเดิมนิดนึง เพราะต้องไป small ก่อน

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> เพราะมันเป็นไปได้ว่าโมเดลที่มีขนาดเล็กจะมั่นใจผิดๆได้ แล้วไม่ได้ไปเช็คกับ Large model เช่นรีวิวแนวประชดประชัน และอาจทำให้เจ้าของร้านเข้าใจสถิติผิดๆ

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 5 | 543 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> รีวิวที่เขียนสั้นๆ จะได้ประโยชน์จาก cache บ่อย โดยรีวิวที่มีประสบการณ์เฉพาะจะไม่ได้ประโยชน์เลย

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> ผลมันก็จะไม่ valid แล้ว ดังนั้นเราควรจะมีการทำเคลียแคชได้ ถ้าเราเพิ่ม version ของโมเดลไปด้วย จะแก้ปัญหา cache ที่มีได้ ส่วนการล้าง cache จะให้เป็น cron job ก็ได้

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> ผมได้เรียนรู้ว่า Quality มันแลกมาด้วยการลดอีก quality นึงบ่อยๆ
