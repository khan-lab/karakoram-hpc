# Installed Modules

All software modules currently available on Karakoram HPC. Filter by name, version, or description. Run `module avail` on the cluster for the authoritative live list.

<input
  type="text"
  id="mod-search"
  placeholder="Filter modules…"
  style="width:100%;padding:.5rem .75rem;margin:.5rem 0 1rem;border:1px solid var(--md-default-fg-color--lightest);border-radius:.2rem;font-size:.85rem;background:var(--md-default-bg-color);color:var(--md-default-fg-color);outline:none;"
/>

<div id="mod-table" markdown="1">

| Tool | Version | Load command | Description |
|------|---------|-------------|-------------|
| alphagenome | 0.6.1 | `module load alphagenome/0.6.1` | Google DeepMind's genomic foundation model for regulatory sequence prediction |
| bcftools | 1.23.1 | `module load bcftools/1.23.1` | Utilities for VCF/BCF variant calling and manipulation |
| bedops | 2.4.42 | `module load bedops/2.4.42` | High-performance set operations and statistics on genomic intervals |
| bedtools | 2.31.1 | `module load bedtools/2.31.1` | Genome arithmetic — intersect, merge, and annotate genomic intervals |
| biopython | 1.70 | `module load biopython/1.70` | Python library for bioinformatics sequence analysis and data parsing |
| blast | 2.17.0 | `module load blast/2.17.0` | Sequence similarity search against nucleotide and protein databases |
| bowtie2 | 2.5.5 | `module load bowtie2/2.5.5` | Fast short-read aligner for DNA sequences |
| bwa | 0.7.19 | `module load bwa/0.7.19` | Burrows-Wheeler Aligner for mapping short reads to a reference genome |
| claude-code | 2.1.123 | `module load claude-code/2.1.123` | Anthropic's Claude Code — AI coding agent CLI for the terminal |
| cnvkit | 0.9.13 | `module load cnvkit/0.9.13` | Copy number variant detection from targeted DNA sequencing |
| codex | 0.125.0 | `module load codex/0.125.0` | OpenAI Codex CLI — AI coding agent for terminal-based development |
| cursor-cli | 2026.04.29 | `module load cursor-cli/2026.04.29` | Cursor AI coding agent for terminal-based AI-assisted development |
| cutadapt | 5.2 | `module load cutadapt/5.2` | Adapter and quality trimming for FASTQ sequencing reads |
| deeptools | 3.5.6 | `module load deeptools/3.5.6` | NGS data normalization, bigWig generation, and heatmap visualization |
| delly | 1.7.3 | `module load delly/1.7.3` | Structural variant discovery using paired-end and split-read signals |
| egafetch | 1.1.0 | `module load egafetch/1.1.0` | Command-line client for downloading data from the European Genome-phenome Archive |
| encodefetch | 0.3.0 | `module load encodefetch/0.3.0` | Download data files from the ENCODE project portal |
| encodefetch | 0.4.0 | `module load encodefetch/0.4.0` | Download data files from the ENCODE project portal |
| fastqc | 0.12.1 | `module load fastqc/0.12.1` | Per-sample quality control report for raw FASTQ sequencing data |
| gatk4 | 4.6.2.0 | `module load gatk4/4.6.2.0` | Genome Analysis Toolkit — variant discovery and genotyping |
| gridss | 2.13.2 | `module load gridss/2.13.2` | Structural variant and genomic rearrangement detection from WGS |
| homer | 5.1 | `module load homer/5.1` | ChIP-seq peak calling, motif discovery, and RNA-seq analysis |
| htslib | 1.23.1 | `module load htslib/1.23.1` | C library for reading and writing SAM, BAM, CRAM, and VCF files |
| intervene | 0.6.4 | `module load intervene/0.6.4` | Venn and UpSet diagram intersection of multiple genomic region sets |
| kallisto | 0.52.0 | `module load kallisto/0.52.0` | Pseudoalignment-based RNA-seq transcript quantification |
| last | 1651 | `module load last/1651` | Many-to-many local sequence alignment, suited to long reads and repeats |
| macs3 | 3.0.4 | `module load macs3/3.0.4` | Peak calling for ChIP-seq and ATAC-seq experiments |
| manta | 1.6.0 | `module load manta/1.6.0` | Structural variant and small indel caller optimized for WGS/WES |
| meme | 5.5.9 | `module load meme/5.5.9` | Motif discovery, enrichment analysis, and database scanning suite |
| mosdepth | 0.3.14 | `module load mosdepth/0.3.14` | Fast sequencing depth calculation from BAM/CRAM files |
| multiqc | 1.34 | `module load multiqc/1.34` | Aggregate QC metrics from many tools and samples into a single report |
| nextflow | 25.10.4 | `module load nextflow/25.10.4` | Scalable and reproducible scientific workflow engine |
| nf-core | 4.0.1 | `module load nf-core/4.0.1` | Community curated Nextflow pipelines and pipeline development tools |
| openjdk | 17.0.18 | `module load openjdk/17.0.18` | OpenJDK Java runtime environment (LTS) |
| openjdk | 24.0.2 | `module load openjdk/24.0.2` | OpenJDK Java runtime environment |
| openjdk | 25.0.2 | `module load openjdk/25.0.2` | OpenJDK Java runtime environment |
| picard | 3.4.0 | `module load picard/3.4.0` | SAM/BAM/CRAM/VCF manipulation and sequencing QC metrics |
| pyega3 | 5.2.0 | `module load pyega3/5.2.0` | Python client for downloading controlled-access datasets from the EGA |
| salmon | 1.11.4 | `module load salmon/1.11.4` | Alignment-free RNA-seq transcript quantification |
| sambamba | 1.0.1 | `module load sambamba/1.0.1` | High-performance BAM sorting, indexing, and duplicate marking |
| samtools | 1.22 | `module load samtools/1.22` | Tools for reading, writing, and manipulating SAM/BAM/CRAM alignments |
| samtools | 1.23.1 | `module load samtools/1.23.1` | Tools for reading, writing, and manipulating SAM/BAM/CRAM alignments |
| scvi | 0.6.8 | `module load scvi/0.6.8` | Deep generative models for single-cell omics data analysis |
| snakemake | 9.19.0 | `module load snakemake/9.19.0` | Python-based workflow management system for reproducible analyses |
| sra-tools | 3.4.1 | `module load sra-tools/3.4.1` | NCBI SRA Toolkit for downloading and converting sequencing data |
| star | 2.7.11b | `module load star/2.7.11b` | Splice-aware RNA-seq aligner with chimeric read detection |
| strelka | 2.9.10 | `module load strelka/2.9.10` | Somatic and germline SNV/indel caller for WGS/WES data |
| telseq | 0.0.2 | `module load telseq/0.0.2` | Telomere length estimation from whole-genome sequencing data |
| tobias | 0.17.3 | `module load tobias/0.17.3` | ATAC-seq transcription factor footprinting and binding site analysis |
| vcftools | 0.1.17 | `module load vcftools/0.1.17` | Tools for filtering, comparing, and summarizing VCF genotype data |

</div>

<script>
(function () {
  var input = document.getElementById('mod-search');
  input.addEventListener('input', function () {
    var q = this.value.toLowerCase();
    var rows = document.querySelectorAll('#mod-table tbody tr');
    rows.forEach(function (row) {
      row.style.display = row.textContent.toLowerCase().indexOf(q) !== -1 ? '' : 'none';
    });
  });
})();
</script>
