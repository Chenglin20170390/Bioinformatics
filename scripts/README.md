# Depth analysis from genome alignment

## quickly stats  depth and covrage (https://github.com/HuiyangYu/PanDepth)
```
pandepth -i test.bam -o test1 -t 36
```
## Depth by bin
- Alignment
```
ref=/home/work/1_reference/DM_v6.1_all_chr.fa
sampe=ID
##illumina 
bwa mem -t $th ${ref} ${seq_1} ${seq_2}| samtools view -@ $th -bS - | samtools sort -@ $th -o $sample.hic.bam -
##or long reads
minimap2 -ax map-hifi -t $th ${ref} ${poreC} | samtools view -@ $th -bS - | samtools sort -@ $th -o $sample.poreC.bam -
```
- make 500k windows for bedtools 
ref=/home/work/1_reference/DM_v6.1_all_chr.fa
```
bedtools makewindows -g $ref.fai -w 50000 > genome_bins.500k.bed
```
- count depth
```
mosdepth --no-per-base --fast-mode  -t $th  -b genome_bins.500k.bed  $sample.poreC $sample.poreC.bam
```

- plot
```
#read input file
data <- read.table("C88.poreC.regions.bed", header=FALSE, sep="\t")
# rename
colnames(data) <- c("chrom", "start",'end', "depth")
head(data)
# pick chrom
chromosomes_to_plot <- unique(data$chrom)[01:12]  # pick chrom
data_filtered <- data %>% filter(chrom %in% chromosomes_to_plot)
data_filtered$start<-data_filtered$end/1000000
data_filtered$depth[data_filtered$depth > 100] <- 50

##plot
p<-ggplot(data_filtered, aes(x=start, y=depth)) +
  geom_area(size=0.8) +
  facet_wrap(~chrom, scales="free_x", ncol=3) +  # chrom
  theme_minimal() +
  labs(title="Chromosome Depth (500k bins)", x="Genomic Position (Mb)", y="Read Depth (x)") +
  theme(legend.position="none") 
p
ggsave(p, filename= "PoreC.depth.pdf" , width=30, height=18,  units =c( "cm" ),colormodel= "srgb" )
```
![depth](https://github.com/user-attachments/assets/bfb3b732-dec1-42d0-9940-98964e5159ba)



