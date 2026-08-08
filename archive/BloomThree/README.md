# This is not to be edited by AI

# What is this about

BloomThree is an application that takes in cancer patient's test results and outputs mRNA that cure their cancer. 

# Application flow description
1. User goes onto the page
2. User reads about the Bloom app
3. User uploads all of their test results
4. User see the test results go thru the 7 stages of analysis
5. User see sharable result - an mRNA card
6. the CTA here are either to share the result card on social media, text etc, or to purchase a piece of equipment for the bloom team to further the ‘cure all cancer’ pipeline

this is the technical description of 7 stages:

Stage 0 · AI Data Extraction
Input: PDF lab reports, screenshots/images, CSV/TSV/Excel, raw text paste
Process: Gemini 2.5 Pro Vision extracts structured data via JSON extraction prompt
Output: mutations[] (gene, hgvsp, vaf, chr, pos, type), hla_alleles[], confidence score
Stage 1 · Data Ingestion
Input: cBioPortal study/sample ID, GDC/TCGA case UUID, VCF/MAF upload, Stage 0 output, or BAM file
Process: Fetches from cBioPortal REST / GDC GraphQL APIs, or parses VCF/MAF files; BAM passes through to Stage 2
Output: Normalized mutations[] (gene, hgvsp, vaf, chr, start, end, ref, alt), mutation_count, hla_alleles[], data_source
Stage 2 · Somatic Mutation Calling
Input: Stage 1 output (mutations + data_source)
Process: If mutations already exist → skip. If BAM source → runs GATK Mutect2 on Modal GPU + gnomAD germline filter
Output: vcf_mutations[] (PASS somatic variants) or skipped: true
Stage 3 · Peptide Generation
Input: mutations[] from Stage 1 or 2
Process: HGVS parser (3-letter → 1-letter AA) → lookup in Ensembl GRCh38 human proteome → sliding window k-mers (8, 9, 10, 11)
Output: peptides[] (sequence, gene, mutation, position, length), peptide_count, genes_processed
Stage 4 · HLA Binding Prediction
Input: peptides[] from Stage 3, hla_alleles[] from patient config
Process: Normalize HLA notation → MHCflurry 2.0 on Modal CPU → classify binders (IC50 <500nM = strong, <5000nM = weak)
Output: binding_results[] (peptide, allele, ic50, percentile_rank), strong_binders, weak_binders, top_binders[] (top 50)
Stage 5 · Safety Filter
Input: Strong binders from Stage 4
Process: Deduplicate peptides → DIAMOND blastp on Modal vs UniProt human proteome → remove ≥80% identity hits (autoimmune risk)
Output: safe_candidates[] (self_similarity, coverage), removed[] (reason, best_hit_protein), safe_count / removed_count
Stage 6 · Candidate Ranking
Input: safe_candidates[] from Stage 5
Process: Min-max normalize → composite score: IC50 binding (50%) + VAF frequency (30%) + TPM expression (20%, optional) → sort top N (default 20)
Output: ranked_candidates[] (composite_score, rank), top_candidates[], score_range
Stage 7 · mRNA Construct Design
Input: top_candidates[] from Stage 6
Process: Deduplicate → reverse translate AA → DNA with human-optimized codon table → assemble: 5'UTR → Kozak → signal peptide → epitope cassette (GGGGSGGGGS linkers) → stop → 3'UTR → poly-A(120)
Output: construct (full mRNA nucleotide sequence), construct_length, gc_content%, epitopes[] (position, peptide, dna), protein_product

# Goal

I have built many different versions of this application in the past and came to the realization that it could be the simplest thing possible. Hence the flow above.

Using the paper.design MCP tool, create for me the design of each screen of the application for me.

Use project 'ComfySpace Design', page 'BloomThree'.

I will be using multiple AI agents to create these screens, so DO NOT peak at others' work in the same page.