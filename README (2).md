![image](https://github.com/user-attachments/assets/cdd8cc7f-562a-443d-9869-6f400654f468)
# BioE 230 - Week 5: Genome Annotation 

Create a README in a Github repository with the answers.

## Question 1 

#### If given the amino acid sequence KVRMFTSELDIMLSVNGPADQIKYFCRHWT*, what is the number of amino acids in the encoded peptide (not including the stop codon)? 

### Command 
```javascript
echo -n "KVRMFTSELDIMLSVNGPADQIKYFCRHWT" | awk '{print length}' 
```


###  Output
```javascript
30 

```
#### Additonally, how many bases are contained in the open reading frame of the DNA sequence encoding the amino acids (including the stop codon)?

### Command 
```javascript
echo $(( ($(echo -n "KVRMFTSELDIMLSVNGPADQIKYFCRHWT" | awk '{print length}') + 1) * 3 ))  
```


###  Output
```javascript
93

```

## Question 2 

#### Run prodigal on one of the genomes you have previously downloaded. Using command line tools, count how many genes were annotated (you can use any of the output formats for this but some are easier than others). 

### Command 
```javascript
prodigal -i GCA_000007125.1_ASM712v1_genomic.fna -o genes.gbk -a proteins.faa -f gbk; grep -c 'CDS' genes.gbk
```


###  Output
```javascript
3152 

```

## Question 3 

#### Run prodigal on all of the genomes you have previously downloaded. Using command line tools, find which genome has the highest number of genes. Put all your code into a shell script, and put your code on the repository on Github where you keep your README with the solutions to this assignment.

### Command to create the shell script and the code 
```javascript
nano run_prodigal.sh
```

```javascript

#!/bin/bash

# Directory containing genome files
GENOME_DIR="/home/abazaiea/ncbi_dataset/prodigal_output/"  # Use the full path
OUTPUT_DIR="/home/abazaiea/ncbi_dataset/prodigal_output/" # Change to desired output path

# Create output directory if it doesn't exist
mkdir -p "$OUTPUT_DIR"

# Variables to track the total number of genes and the genome with the highest number of genes
total_genes=0  # Initialize a variable to keep track of the total number of genes across all genomes
max_genes=0    # Initialize a variable to keep track of the maximum number of genes found in a single genome
best_genome=""  # Initialize a variable to store the name of the genome with the highest number of genes

# Loop through all fasta files in the genome directory
for genome in "$GENOME_DIR"/*.fna; do
    # Check if the file exists
    if [[ ! -e "$genome" ]]; then
        echo "No fasta files found in $GENOME_DIR."
        exit 1
    fi

    # Run Prodigal
    prodigal -i "$genome" -o "$OUTPUT_DIR/$(basename "$genome" .fna).gbk" -a "$OUTPUT_DIR/$(basename "$genome" .fna).faa" -f gbk
    
    # Count the number of genes
    gene_count=$(grep -c 'CDS' "$OUTPUT_DIR/$(basename "$genome" .fna).gbk")
    
    # Update total gene count
    total_genes=$((total_genes + gene_count))  # Add the count of genes for the current genome to the total gene count

    # Print the gene count for the current genome
    echo "Genome: $(basename "$genome"), Genes: $gene_count"

    # Check if this genome has the highest number of genes
    if [[ "$gene_count" -gt "$max_genes" ]]; then
        max_genes=$gene_count  # Update max_genes if the current genome has more genes
        best_genome=$(basename "$genome")  # Store the name of the genome with the highest gene count
    fi
done

# Output the total number of genes and the genome with the highest number of genes
if [[ "$total_genes" -eq 0 ]]; then
    echo "No genes were found in any genomes."
else
    echo "Total number of genes across all genomes: $total_genes"
    echo "Genome with the highest number of genes: $best_genome with $max_genes genes"
fi


```

### Commands to run prodigal 
```javascript
chmod +x run_prodigal.sh
./run_prodigal.sh
```


###  Output
```javascript
Genome: GCA_000006745.1_ASM674v1_genomic.fna, Genes: 3594
Genome: GCA_000006825.1_ASM682v1_genomic.fna, Genes: 2032
Genome: GCA_000006865.1_ASM686v1_genomic.fna, Genes: 2383
Genome: GCA_000007125.1_ASM712v1_genomic.fna, Genes: 3152
Genome: GCA_000008525.1_ASM852v1_genomic.fna, Genes: 1579
Genome: GCA_000008545.1_ASM854v1_genomic.fna, Genes: 1866
Genome: GCA_000008565.1_ASM856v1_genomic.fna, Genes: 3248
Genome: GCA_000008605.1_ASM860v1_genomic.fna, Genes: 1009
Genome: GCA_000008625.1_ASM862v1_genomic.fna, Genes: 1776
Genome: GCA_000008725.1_ASM872v1_genomic.fna, Genes: 897
Genome: GCA_000008745.1_ASM874v1_genomic.fna, Genes: 1063
Genome: GCA_000008785.1_ASM878v1_genomic.fna, Genes: 1505
Genome: GCA_000027305.1_ASM2730v1_genomic.fna, Genes: 1748
Genome: GCA_000091085.2_ASM9108v2_genomic.fna, Genes: 1063

```

```javascript
Total number of genes across all genomes: 26915
Genome with the highest number of genes: GCA_000006745.1_ASM674v1_genomic.fna with 3594 genes

```

## Question 4 

#### Annotate all genomes you have previously downloaded using prokka instead of prodigal. Using shell commands, count the number of coding sequences (CDS) annotated by Prokka. Are the total number of genes the same as they were with prodigal? What are the differences?

### Command to create the shell script and the code 
```javascript
nano run_prokka.sh
```

```javascript

#!/bin/bash

# Directory containing genome files
GENOME_DIR="/home/abazaiea/ncbi_dataset/data"  # Input path where genome files are located
OUTPUT_DIR="/home/abazaiea/ncbi_dataset/prokka_output" # Output path where Prokka results will be saved

# Create output directory if it doesn't exist
mkdir -p "$OUTPUT_DIR"  # Creates the output directory if it doesn't already exist

# Variables to track the total number of CDS and the genome with the highest number of CDS
total_cds=0  # Initialize a variable to keep track of the total number of CDS across all genomes
max_cds=0    # Initialize a variable to keep track of the maximum number of CDS found in a single genome
best_genome=""  # Initialize a variable to store the name of the genome with the highest number of CDS

# Loop through all fasta files in the genome directory
for genome in "$GENOME_DIR"/*.fna; do
    # Check if the file exists
    if [[ ! -e "$genome" ]]; then
        echo "No fasta files found in $GENOME_DIR."  # Inform the user if no fasta files are found
        exit 1  # Exit the script with an error status
    fi

    # Run Prokka
    prokka --outdir "$OUTPUT_DIR/$(basename "$genome" .fna)" --prefix "$(basename "$genome" .fna)" "$genome" --force
    # Execute Prokka to annotate the genome

    # Count the number of CDS in the .gbk file
    cds_count=$(grep -c '^     CDS' "$OUTPUT_DIR/$(basename "$genome" .fna)/$(basename "$genome" .fna).gbk")
    
    # Update total CDS count
    total_cds=$((total_cds + cds_count))  # Add the count of CDS for the current genome to the total CDS count

    # Print the CDS count for the current genome
    echo "Genome: $(basename "$genome"), CDS: $cds_count"

    # Check if this genome has the highest number of CDS
    if [[ "$cds_count" -gt "$max_cds" ]]; then
        max_cds=$cds_count  # Update max_cds if the current genome has more CDS
        best_genome=$(basename "$genome")  # Store the name of the genome with the highest CDS
    fi
done

# Output the total number of CDS and the genome with the highest number of CDS
if [[ "$total_cds" -eq 0 ]]; then
    echo "No CDS were found in any genomes."  # Inform the user if no CDS were found
else
    echo "Total number of CDS across all genomes: $total_cds"  # Display the total CDS count
    echo "Genome with the highest number of CDS: $best_genome with $max_cds CDS"  # Display the best genome info
fi

```

### Commands to run prokka 
```javascript
chmod +x run_prokka.sh
./run_prokka.sh
```

###  Output
```javascript
Genome: GCA_000006745.1_ASM674v1_genomic.fna,  Genes: 3589
Genome: GCA_000006825.1_ASM682v1_genomic.fna,  Genes:2028
Genome: GCA_000006865.1_ASM686v1_genomic.fna, Genes: 2383
Genome: GCA_000007125.1_ASM712v1_genomic.fna, Genes: 3150
Genome: GCA_000008525.1_ASM852v1_genomic.fna, Genes: 1577
Genome: GCA_000008545.1_ASM854v1_genomic.fna, Genes: 1861
Genome: GCA_000008565.1_ASM856v1_genomic.fna, Genes: 3245
Genome: GCA_000008605.1_ASM860v1_genomic.fna, Genes: 1001
Genome: GCA_000008625.1_ASM862v1_genomic.fna, Genes: 1771
Genome: GCA_000008725.1_ASM872v1_genomic.fna, Genes: 892
Genome: GCA_000008745.1_ASM874v1_genomic.fna, Genes: 1058
Genome: GCA_000008785.1_ASM878v1_genomic.fna, Genes: 1504
Genome: GCA_000027305.1_ASM2730v1_genomic.fna, Genes:1748
Genome: GCA_000091085.2_ASM9108v2_genomic.fna, Genes: 1056

```

```javascript
Total number of CDS across all genomes: 26863
Genome with the highest number of CDS: GCA_000006745.1_ASM674v1_genomic.fna with 3589

```

- The difference is that prodigal is faster than prokka and the total gene count is also higher.  
    26915 > 26863
 


## Question 5 

#### Extract and list all unique gene names annotated by Prokka using shell commands. Provide the command you used and the first five gene names from the list.

### Command to extract the list of unique genes 
```javascript
grep -h "/gene=" *.gbk | cut -d'=' -f2 | tr -d '"' | sort | uniq > unique_genes.txt
```

### Command to show the fist five names 
```javascript
head -n 5 unique_genes.txt
```

###  Output 
```javascript
aaaT
aaeA
aaeA_1 
aaeA_2
aaeB

```
