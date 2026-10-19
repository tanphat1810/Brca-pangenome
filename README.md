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

```bash
fastp \
      -i "$r1_path" \
      -I "$r2_path" \
      -o "${OUTPUT_DIR}clean_fastq/${sample_name}_clean_R1.fastq.gz" \
      -O "${OUTPUT_DIR}clean_fastq/${sample_name}_clean_R2.fastq.gz" \
      --trim_front1 30 \
      --trim_front2 30 \
      --detect_adapter_for_pe \
      --cut_right \
      --cut_right_window_size 4 \
      --cut_right_mean_quality 20 \
      --qualified_quality_phred 20 \
      --unqualified_percent_limit 20 \
      --length_required 50 \
      --correction \
      --thread 8 \
      --html "${OUTPUT_DIR}reports/${sample_name}.html" \
      --json "${OUTPUT_DIR}reports/${sample_name}.json"
```

#### 3.1.3. Kiểm tra dữ liệu sau khi lọc

```bash
fastqc Fastp/clean_fastq/*.gz -o FastQCFiltered -t 8
multiqc FastQCFiltered -o MultiQCFiltered
```
#### 3.1.4. Mapping

```bash
./mapping.sh -r <Đường_dẫn_file_Ref> -i <Thư_mục_chứa_Fastq> -o <Thư_mục_kết_quả>
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

```bash
fastp \
      -i "$r1_path" \
      -I "$r2_path" \
      -o "${OUTPUT_DIR}clean_fastq/${sample_name}_clean_R1.fastq.gz" \
      -O "${OUTPUT_DIR}clean_fastq/${sample_name}_clean_R2.fastq.gz" \
      --detect_adapter_for_pe \
      --trim_front1 30 \
      --trim_front2 30 \
      --qualified_quality_phred 20 \
      --unqualified_percent_limit 20 \
      --length_required 50 \
      --correction \
      --thread 8 \
      --html "${OUTPUT_DIR}reports/${sample_name}.html" \
      --json "${OUTPUT_DIR}reports/${sample_name}.json"
```

#### 3.1.3. Kiểm tra dữ liệu sau khi lọc

```bash
fastqc Fastp/clean_fastq/*.gz -o FastQCFiltered -t 8
multiqc FastQCFiltered -o MultiQCFiltered
```
