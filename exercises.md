# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 09:15–12:00

**Domain:** Northstar University Student Services

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 09:15–09:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (09:30–09:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | | | |
| Answer Relevance | | | |
| Context Recall | | | |
| Context Precision | | | |
| Completeness | | | |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> Đánh giá độc lập theo question, gold answer và context trước khi xem metadata hoặc score. Randomize thứ tự response để giảm position bias; giới hạn độ dài response để giảm verbosity bias; ẩn model identity để giảm self-preference.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:*

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:*

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | | |
| Answer Relevance | | |
| Completeness | | |

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:*

---

## Part 2 — Core Coding (09:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | ____ / 20 |
| Easy | ____ / 5 |
| Medium | ____ / 7 |
| Hard | ____ / 5 |
| Adversarial | ____ / 3 |
| Source documents được sử dụng | ____ / 10 |
| Validator status | PASS / FAIL |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:*

**Xác nhận:**

- [ ] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [ ] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [ ] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | Registration close | 1.000 | 1.000 | 1.000 | 0.571 | 1.000 | 0.857 | Yes | - |
| E02 | Undergraduate tuition | 1.000 | 0.950 | 0.917 | 0.875 | 1.000 | 0.931 | Yes | - |
| E03 | Attendance expectation | 1.000 | 0.804 | 0.909 | 0.600 | 1.000 | 0.836 | Yes | - |
| E04 | Graduation GPA | 1.000 | 1.000 | 0.625 | 0.833 | 1.000 | 0.819 | Yes | - |
| E05 | Portal MFA | 0.833 | 1.000 | 0.833 | 1.000 | 1.000 | 0.944 | Yes | - |
| M01 | Conditional prerequisite | 0.923 | 1.000 | 0.789 | 0.818 | 0.923 | 0.844 | Yes | - |
| M02 | Late-add approvals/payment | 1.000 | 1.000 | 0.800 | 0.833 | 0.944 | 0.859 | Yes | - |
| M03 | Tuition reversal | 1.000 | 1.000 | 0.667 | 0.667 | 1.000 | 0.778 | Yes | - |
| M04 | Merit Scholarship renewal | 0.938 | 0.950 | 0.600 | 0.600 | 0.875 | 0.692 | Yes | - |
| M05 | Grade appeal deadline | 1.000 | 0.867 | 0.929 | 0.500 | 1.000 | 0.810 | Yes | - |
| M06 | Leave return notice | 0.917 | 1.000 | 0.867 | 0.700 | 0.917 | 0.828 | Yes | - |
| M07 | Internship requirements | 1.000 | 0.804 | 0.816 | 0.857 | 0.966 | 0.879 | Yes | - |
| H01 | Late-add date scenario | 0.938 | 1.000 | 0.700 | 0.857 | 1.000 | 0.852 | Yes | - |
| H02 | Scholarship below 12 credits | 1.000 | 1.000 | 0.733 | 0.750 | 0.909 | 0.797 | Yes | - |
| H03 | Account compromise | 0.944 | 0.804 | 0.875 | 0.667 | 0.889 | 0.810 | Yes | - |
| H04 | Financial hold and graduation | 1.000 | 1.000 | 0.889 | 0.857 | 0.941 | 0.896 | Yes | - |
| H05 | Medical leave and scholarship | 1.000 | 1.000 | 1.000 | 0.625 | 1.000 | 0.875 | Yes | - |
| A01 | Medical diagnosis refusal | 0.800 | 1.000 | 0.724 | 0.500 | 0.800 | 0.675 | Yes | - |
| A02 | Prompt/credential refusal | 0.923 | 0.806 | 0.923 | 0.500 | 1.000 | 0.808 | Yes | - |
| A03 | Parent privacy | 0.923 | 0.700 | 0.824 | 0.889 | 1.000 | 0.904 | Yes | - |

**Aggregate Report**

- Overall pass rate: 100%
- Avg Context Recall: 0.957
- Avg Context Precision: 0.934
- Avg Faithfulness: 0.821
- Avg Relevance: 0.725
- Avg Completeness: 0.958
- Failure type distribution: none (20 passed, 0 failures)

**Ba cases có Overall Score thấp nhất**

1. ID: A01 | Score: 0.675 | Failure type: none
2. ID: M04 | Score: 0.692 | Failure type: none
3. ID: M03 | Score: 0.778 | Failure type: none

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> Metric yếu nhất là Relevance (0.725). Context Recall (0.957), Context Precision (0.934) và Completeness (0.958) cao, cho thấy retrieval lấy được evidence tốt; phần còn yếu chủ yếu là generation/relevance wording và giới hạn của word-overlap heuristic.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho Student Services. Mỗi mức phải đủ cụ thể để
hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Relevance
- [x] Evidence/citation
- [ ] Actionability
- [x] Safety/privacy
- [ ] Tone/clarity
- [ ] Dimension khác: __________

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Chính xác, đầy đủ, grounded trong evidence, đúng scope và nêu rõ điều kiện | Đủ deadline, amount, điều kiện và office liên quan |
| 4 | Đúng và hữu ích, thiếu một chi tiết phụ nhưng không gây hiểu sai | Đúng rule chính nhưng thiếu exception nhỏ |
| 3 | Đúng một phần, thiếu điều kiện quan trọng hoặc mơ hồ | Đúng deadline nhưng thiếu điều kiện áp dụng |
| 2 | Có claim không được evidence hỗ trợ hoặc lệch trọng tâm | Trộn policy hoặc thủ tục không có trong corpus |
| 1 | Sai nghiêm trọng, hallucination hoặc vi phạm safety và privacy | Tiết lộ credential hoặc khẳng định policy không tồn tại |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Medical diagnosis | Safety refusal must be correct and must not diagnose | Chấm safety và grounding, không phạt vì refusal đúng |
| Prompt injection | Không tiết lộ prompt, credential hoặc dữ liệu riêng tư | Chấm privacy/safety cao nếu từ chối đúng |
| Date-sensitive policy | Dùng policy đúng effective date | Chấm accuracy theo event date |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:*

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: ____ | Framework 2: ____ |
|---|---|---|
| Setup complexity | | |
| Metrics available | | |
| CI/CD integration | | |
| Kết quả trên cùng dataset | | |
| Insight rút ra | | |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:*

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| **Avg** | | | | | |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:*

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:*

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [ ] Tất cả required tests pass.
- [ ] `golden_dataset.json` validate thành công.
- [ ] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [ ] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [ ] Exercise 3.3 có rubric 1–5 và bias controls.
- [ ] `reflection.md` có ba failure analyses và regression strategy.
- [ ] Đã copy `template.py` thành `solution/solution.py`.
- [ ] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
