# Lab 21 — Evaluation Report

**Học viên**: LÊ TÚ NAM 
**Ngày nộp**: 2026-05-07  
**Submission option**: B (GitHub + HuggingFace Hub )

---

# 1. Setup

- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit"`
- **Dataset**: Custom Vietnamese Alpaca Dataset, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024 (p95 = 53, rounded up manually for training stability)
- **GPU**: NVIDIA Tesla T4, 15 GB VRAM
- **Training cost**: ~$0.12 (~18 phút @ ~$0.40/hr)
- **HF Hub link**: [Không sử dụng (Option A)](https://huggingface.co/namleeee/qwen2.5-3b-vi-lab21-r16)

---

# 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | ~2.6M           | 5 min      | 7.1 GB    | 2.41      | 11.14      |
| 16   | ~5.2M           | 6 min      | 7.8 GB    | 2.19      | 8.93       |
| 64   | ~20.9M          | 9 min      | 10.4 GB   | 2.11      | 8.25       |
| Base | -               | -          | -         | 3.87      | 47.94      |

---

# 3. Loss Curve Analysis

<img width="1079" height="488" alt="image" src="https://github.com/user-attachments/assets/5d86bc45-4dd8-45b8-b21d-c8e3f77f3f03" />


- Quan sát:
Loss giảm khá nhanh trong giai đoạn đầu và bắt đầu ổn định sau khoảng vài trăm steps. Không xuất hiện overfitting rõ ràng vì dataset khá nhỏ và số epoch không quá lớn. Tuy nhiên ở rank 64, loss improvement bắt đầu chậm lại dù số lượng trainable parameters tăng đáng kể. Điều này cho thấy dataset hiện tại không đủ lớn để tận dụng fully capacity của rank cao.

---

# 4. Qualitative Comparison (5 examples)

## Example 1

**Prompt**: Liệt kê 3 ưu điểm của Python.

**Base**:
Output thiếu ổn định, đôi lúc sinh token rác hoặc câu không liên quan.

**Fine-tuned (r=16)**:
1. Python dễ học và dễ đọc.  
2. Có hệ sinh thái thư viện lớn.  
3. Hỗ trợ tốt cho AI và Data Science.

**Nhận xét**: Improved

---

## Example 2

**Prompt**: Đưa ra 3 ứng dụng của Machine Learning.

**Base**:
Câu trả lời ngắn, không đầy đủ và đôi khi lặp từ.

**Fine-tuned (r=16)**:
1. Hệ thống gợi ý sản phẩm.  
2. Nhận diện khuôn mặt.  
3. Dự đoán dữ liệu tài chính.

**Nhận xét**: Improved

---

## Example 3

**Prompt**: Giải thích AI khác Machine Learning như thế nào.

**Base**:
Giải thích mơ hồ và thiếu cấu trúc.

**Fine-tuned (r=16)**:
AI là lĩnh vực rộng nhằm tạo hệ thống thông minh, còn Machine Learning là một nhánh của AI cho phép máy học từ dữ liệu.

**Nhận xét**: Improved

---

## Example 4

**Prompt**: Deep Learning là gì?

**Base**:
Output ngắn và đôi khi không đúng ngữ cảnh.

**Fine-tuned (r=16)**:
Deep Learning là một nhánh của Machine Learning sử dụng mạng neural nhiều tầng để học đặc trưng dữ liệu phức tạp.

**Nhận xét**: Improved

---

## Example 5

**Prompt**: Ưu và nhược điểm của cloud computing.

**Base**:
Nội dung thiếu rõ ràng.

**Fine-tuned (r=16)**:
Ưu điểm:
- Dễ mở rộng tài nguyên.
- Giảm chi phí hạ tầng.

Nhược điểm:
- Phụ thuộc internet.
- Có rủi ro bảo mật dữ liệu.

**Nhận xét**: Improved

---

# 5. Conclusion về Rank Trade-off

Trong thí nghiệm này, rank 16 cho ROI tốt nhất giữa chất lượng và tài nguyên sử dụng. So với rank 8, rank 16 cải thiện rõ rệt về perplexity và chất lượng output nhưng không tăng VRAM quá nhiều. Trong khi đó rank 64 chỉ cải thiện nhẹ perplexity nhưng thời gian train và memory usage tăng đáng kể.

Hiện tượng diminishing returns bắt đầu xuất hiện từ rank 16 lên rank 64. Dataset chỉ có 200 samples nên model không đủ dữ liệu để tận dụng toàn bộ số parameters bổ sung ở rank cao. Điều này cho thấy việc tăng rank không phải lúc nào cũng mang lại hiệu quả tương ứng.

Nếu deploy production cho bài toán nhỏ hoặc dataset domain-specific nhẹ, tôi sẽ chọn rank 16. Nó đạt cân bằng tốt giữa quality, tốc độ train, VRAM usage và inference efficiency. Rank 64 phù hợp hơn khi có dataset lớn và cần expressive capacity cao hơn.

---

# 6. What I Learned

- LoRA rank ảnh hưởng trực tiếp tới trade-off giữa chất lượng và tài nguyên GPU.
- Prompt formatting rất quan trọng, đặc biệt với Gemma models.
- Dataset nhỏ có thể fine-tune hiệu quả với PEFT mà không cần full fine-tuning.
