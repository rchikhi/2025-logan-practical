## Part A: Accessing Logan data

### 0. Preparation

You need the AWS command-line interface (CLI) installed either on your computer or in a virtual environment (if one is provided by the tutorial).

To check if it is installed, run the command in a terminal.

```
   aws help
```

If it is not installed, you may download it from https://aws.amazon.com/cli/

### 1. Exploring an SRA accession

The Short Read Archive is the central repository of all public sequencing data. Access it at this location: https://www.ncbi.nlm.nih.gov/sra

![image](https://github.com/user-attachments/assets/a0a73965-5764-4dff-acba-71645dde48fc)

We will focus on E. coli long-term evolution run *SRR2584403*, because it is small enough to test.

![image](https://github.com/user-attachments/assets/a4e931d7-35f5-420c-b2b4-1ef138ca9900)

You may see some more information about the run (SRR2584403) by clicking on it. Head to the `Data access` tab to see how to download that run.

![image](https://github.com/user-attachments/assets/d9e6c218-cb9b-4cf3-88e7-dc39a7f83cff)


Now, how to download a FASTQ file? It is not immediate. The way to get a FASTQ file is to first get a SRA file and then convert that SRA file to FASTQ.

To get the SRA file, look at the `SRA archive data` section and download any of the files.

![image](https://github.com/user-attachments/assets/0567c726-37f1-41bb-ade7-64c59df6eddc)

You may download using the AWS CLI by typing:

    wget https://sra-pub-run-odp.s3.amazonaws.com/sra/SRR2584403/SRR2584403
    mv SRR2584403 SRR2584403.sra

Then, what you have is a SRA file that needs to be converted to FASTQ. To do that, install the SRA toolkit https://github.com/ncbi/sra-tools/wiki/02.-Installing-SRA-Toolkit
and type:

   fasterq-dump SRR2584403.sra

If all went well, you should see files `SRR2584403_1.fastq` and `SRR2584403_2.fastq` in your folder, around 700 MB each.

You may attempt to download the data in the `Original format`, at location:

    s3://sra-pub-src-8/SRR2584403/Ara-1_500gen_762B_R1.fastq.gz

by typing:

	  aws s3 cp s3://sra-pub-src-8/SRR2584403/Ara-1_500gen_762B_R1.fastq.gz .

But you'll get an error 403: Forbidden. Turns out the way to access this data is through `Cloud Data Delivery` which requires NCBI to retrieve it from cold storage, so it is not immediate.
