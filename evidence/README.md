# Evidence — Day 22 Lab (Lê Phan Việt Cường — 2A202602641)

**LangSmith project (`day22-lab`, ≥ 100 traces):** https://smith.langchain.com/o/ae060a05-61eb-410b-8545-344cf2cfb29c/projects/p/8bbed069-fd6f-484e-a1a4-5ad9a574d456

**Tổng traces:** 603 traces trong project (`00_total_traces.png`), gồm 50 `rag-query` (`01_langsmith_traces.png`), 100 `ab-rag-query` (`02_ab_traces.png`) và các trace sinh ra khi chạy Bước 3 (RAGAS).

## Danh sách tệp

| Tệp | Nội dung |
|---|---|
| `01_langsmith_traces.png` | Danh sách traces `rag-query` (lọc `tag:step1`) — 50 traces |
| `01_trace_detail.png` | Cây xử lý 1 trace: `VectorStoreRetriever` → `format_docs` → `ChatPromptTemplate` → `ChatOpenAI` → `StrOutputParser` |
| `01_rag_pipeline_log.txt` | Log chạy 50 câu hỏi Bước 1 |
| `00_total_traces.png` | Tab Traces của project `day22-lab`, không lọc — Stats · 603 traces |
| `00_project_overview.png` | Project `day22-lab` trên LangSmith (≥ 100 traces) |
| `02_prompt_hub.png` | 2 prompt `le-phan-viet-cuong-rag-prompt-v1` / `-v2` trên Prompt Hub |
| `02_ab_traces.png` | Danh sách traces `ab-rag-query` (lọc `tag:step2`) — 100 traces (Bước 2 chạy 2 lần: chạy riêng + qua `run_all.py`) |
| `02_ab_routing_log.txt` | Log push/pull Hub + A/B routing có nhãn `[prompt-v1]` / `[prompt-v2]` (V1=19, V2=31) |
| `03_ragas_scores.png` | Bảng so sánh RAGAS V1 vs V2 |
| `03_ragas_report.json` | Bản sao `data/ragas_report.json` |
| `03_ragas_log.txt` | Log đầy đủ lần chạy RAGAS |
| `04_pii_demo_log.txt`, `04_json_demo_log.txt` | Demo PIIDetector (6 case) và JSONFormatter (5 case) |

## Kết quả RAGAS (gpt-4o-mini, 50 cặp QA, FAISS k=3, chunk 500/50)

| Metric | V1 (ngắn gọn) | V2 (chuyên gia, có cấu trúc) | Winner |
|---|---|---|---|
| faithfulness | **0.9595** | 0.9246 | V1 |
| answer_relevancy | **0.9125** | 0.8777 | V1 |
| context_recall | 1.0000 | 1.0000 | Hòa |
| context_precision | 0.9483 | 0.9483 | Hòa |

Cả 2 phiên bản đều đạt faithfulness ≥ 0.9.

## Phân tích: vì sao V1 cao hơn V2

1. **Faithfulness — V1 thắng vì ít "claim" hơn.** RAGAS tách câu trả lời thành các mệnh đề rồi kiểm tra từng mệnh đề có được context hỗ trợ không. V1 giới hạn 2–4 câu và chỉ nêu ý chính, nên hầu hết mệnh đề lấy thẳng từ context. V2 yêu cầu 3–5 câu và "giải thích cơ chế hoặc chi tiết quan trọng", khiến model dễ diễn giải thêm, khái quát hóa hoặc bổ sung kiến thức nền không có nguyên văn trong 3 chunk được truy xuất — mỗi mệnh đề như vậy làm giảm điểm, dù prompt V2 đã dặn "không suy diễn thêm".

2. **Answer relevancy — V1 thắng vì tập trung hơn.** Chỉ số này sinh lại câu hỏi từ câu trả lời và đo độ tương đồng với câu hỏi gốc. Câu trả lời ngắn, đi thẳng vào trọng tâm sinh ra câu hỏi gần với câu gốc; câu trả lời dài, có phần mở rộng (định nghĩa + cơ chế + chi tiết) kéo embedding lệch sang các ý phụ.

3. **Context recall / precision — bằng nhau là đúng kỳ vọng.** Hai chỉ số này chỉ phụ thuộc vào context được truy xuất và đáp án chuẩn, không phụ thuộc câu trả lời. Cả V1 và V2 dùng chung retriever (cùng FAISS index, k=3) nên nhận đúng cùng 3 chunk cho mỗi câu hỏi. context_recall = 1.0 cho thấy kho tài liệu và chunking hiện tại đủ bao phủ toàn bộ đáp án chuẩn.

**Kết luận:** với bộ câu hỏi dạng định nghĩa/factual như lab này, prompt ngắn gọn (V1) vừa bám tài liệu hơn vừa đúng trọng tâm hơn. V2 phù hợp hơn khi người dùng cần giải thích sâu, nhưng muốn giữ faithfulness cao cần ràng buộc chặt hơn, ví dụ yêu cầu mỗi câu phải dựa trên một đoạn context cụ thể.
