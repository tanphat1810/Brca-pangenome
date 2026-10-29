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
./Fastp.sh Raw Fastp
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
./MappingBrca.sh -r <Đường_dẫn_file_Ref> -i <Thư_mục_chứa_Fastq> -o <Thư_mục_kết_quả>
```
#### 3.1.5. Kiem tra sau mapping

```bash
python Script/CheckMapping.py \
    -i MappingBCRA \
    -o MappingBCRA/QCSummary \
    --good 90 \
    --warn 80 \
    --quickcheck
```

#### 3.1.6. Variant calling
```bash
python Script/brca_graph_pipeline.py \
    --bam-dir Mapping/BAM \
    --reference BrcaRef/BRCA_combined.fa \
    --flagstat-dir Mapping/Stats \
    --min-mapping-pct 90 \
    --outdir BRCA_Pangenome \
    --threads 16 \
    --min-mapq 20 \
    --min-baseq 20 \
    --max-depth 10000 \
    --min-qual 30 \
    --min-dp 10 \
    --quickcheck \
    --index-giraffe
```

#### 3.1.7. Kiểm tra kết quả variant calling

Đếm VCF sau lọc:
```bash
find Pangenome/per_sample \
    -name "*.filtered.vcf.gz" \
    -type f | wc -l
```

Kiểm tra file và index:
```bash
python - <<'PY'
from pathlib import Path

lst = Path("BRCA_Pangenome/cohort/vcf_files.txt")
files = [
    Path(x.strip())
    for x in lst.read_text().splitlines()
    if x.strip()
]

missing = [p for p in files if not p.is_file()]

missing_index = [
    p for p in files
    if not Path(str(p) + ".csi").is_file()
    and not Path(str(p) + ".tbi").is_file()
]

print("Total VCF:", len(files))
print("Missing VCF:", len(missing))
print("Missing index:", len(missing_index))
PY
```

#### 3.1.8. Gộp VCF thành cohort VCF

merge:
```bash
bcftools merge \
    -m none \
    --threads 16 \
    -l Pangenome/cohort/vcf_files.txt \
    -Oz \
    -o Pangenome/cohort/cohort.merged.vcf.gz
```
Tạo index:

```bash
bcftools index -f \
    Pangenome/cohort/cohort.merged.vcf.gz
```

#### 3.1.9. Hợp nhất các allele tại cùng vị trí
```bash
bcftools norm \
    -f BrcaRef/BRCA_combined.fa \
    -m +any \
    -Oz \
    -o Pangenome/cohort/cohort.multiallelic.vcf.gz \
    BRCA_Pangenome/cohort/cohort.merged.vcf.gz
```

Tạo index:
```bash
bcftools index -f \
    Pangenome/cohort/cohort.multiallelic.vcf.gz
```

#### 3.1.10. Xây dựng BRCA variation graph và Giraffe indexes

Tạo thư mục graph:
```bash
mkdir -p Pangenome/graph
```

Chạy vg autoindex:
```bash
set -o pipefail

vg autoindex \
    --workflow giraffe \
    -r BrcaRef/BRCA_combined.fa \
    -v Pangenome/cohort/cohort.multiallelic.vcf.gz \
    -p Pangenome/graph/BRCA_graph \
    -t 16 \
    2>&1 | tee Pangenome/graph/autoindex.log
```

#### 3.1.11. Chuyển GBZ sang GFA
```bash
vg convert -fW \
    Pangenome/graph/BRCA_graph.giraffe.gbz \
    > Pangenome/graph/BRCA_graph.gfa
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
./Fastp.sh Raw Fastp
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
