# Brca-pangenome
## I. Tóm tắt
Quy trình phân tích Pangenome cho đoạn gen Brca 1 và Brca 2 nhằm tăng chất lượng xử lý

## II. Cấu trúc thư mục

## III. Mô tả chi tiết quy trình xử lý 
### 3.1. Thí nghiệm 1
Gộp 2 gen Brca 1 và Brca 2 thành 1 đoạn

#### 3.1.1. Kiểm tra chất lượng dữ liệu thô (FastQC, MultiQC)
```bash
fastqc Raw/*.gz -o FastQC -t 8
multiqc FastQC -o MultiQC
```

#### 3.1.2. Làm sạch reads (fastp)
```bash
./run_fastp.sh Raw Fastp
```

### 3.1. Thí nghiệm 2
Phân tích 2 gen Brca 1 và Brca 2 riêng lẻ

#### 3.1.1. Kiểm tra chất lượng dữ liệu thô (FastQC, MultiQC)
```bash
fastqc Raw/*.gz -o FastQC -t 8
multiqc FastQC -o MultiQC
```

#### 3.1.2. Làm sạch reads (fastp)
```bash
./run_fastp.sh Raw Fastp
```
