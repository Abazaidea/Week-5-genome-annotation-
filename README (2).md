
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
GENOME_DIR="/home/abazaiea/ncbi_dataset/data"  # input path
OUTPUT_DIR="/home/abazaiea/ncbi_dataset/data" # output path

# Create output directory if it doesn't exist
mkdir -p "$OUTPUT_DIR"

# Variables to track the genome with the highest number of genes
max_genes=0
best_genome=""

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
    
    echo "Genome: $(basename "$genome"), Genes: $gene_count"

    # Check if this genome has the highest number of genes
    if [[ "$gene_count" -gt "$max_genes" ]]; then
        max_genes=$gene_count
        best_genome=$(basename "$genome")
    fi
done

if [[ "$max_genes" -eq 0 ]]; then
    echo "No genes were found in any genomes."
else
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
Total number of genes across all genomes: 53824
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
OUTPUT_DIR="/home/abazaiea/ncbi_dataset/data" # Output path where Prokka results will be saved

# Create output directory if it doesn't exist
mkdir -p "$OUTPUT_DIR"  # Creates the output directory if it doesn't already exist, suppressing errors if it does

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
    # Execute Prokka to annotate the genome. The output files will be saved in a directory named after the genome,
    # and the prefix for the output files will also match the genome's name. The '--force' option allows overwriting existing files.

    # Count the number of CDS in the .gbk file
    cds_count=$(grep -c '^     CDS' "$OUTPUT_DIR/$(basename "$genome" .fna)/$(basename "$genome" .fna).gbk")
    # Use grep to count the number of lines that start with '     CDS' in the .gbk file, indicating the presence of CDS annotations.

    # Update total CDS count
    total_cds=$((total_cds + cds_count))  # Add the count of CDS for the current genome to the total CDS count

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
 Total number of CDS across all genomes: 53722
Genome with the highest number of CDS: GCA_000006745.1_ASM674v1_genomic.fna with 3589

```

- The difference is that prodigal is faster than prokka and the total gene count is also higher.  
    53824 > 53722  --> 26912 > 26861
- The number is accounting for all 28 files present in the folder, so the numbers should be divided by 2 like shown. 




## Question 5 

#### Extract and list all unique gene names annotated by Prokka using shell commands. Provide the command you used and the first five gene names from the list.

### Command to extract the list of unique genes 
```javascript
cat *.tsv | cut -f 4 | grep -v '^$' | uniq > unique_genes.txt  
```

### Command to show the fist five names 
```javascript
head -n 5 unique_genes.txt
```

###  Output 
```javascript
gene
mioC
mnmE
yidC
rnpA

```