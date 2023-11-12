# eQTL Catalogue
## Install transfer utility
```bash
sudo apt install aria2
```

## Copy summary statistics
```bash
wget -qO- "https://raw.githubusercontent.com/eQTL-Catalogue/eQTL-Catalogue-resources/master/tabix/tabix_ftp_paths_imported.tsv" \
  | tail -n+2 \
  | cut -f8 \
  > input_filenames.txt

time aria2c -i input_filenames.txt

function upload_file () {
  export FULL_FILENAME=$1
  export LOCAL_FILENAME=$2
  SUFFIX=$(echo $FULL_FILENAME | sed -e 's@ftp://ftp.ebi.ac.uk/pub/databases/spot/eQTL/@@')
  GS_OUTFILE="gs://genetics_etl_python_playground/input/preprocess/eqtl_catalogue/$SUFFIX"
  echo gsutil cp $LOCAL_FILENAME $GS_OUTFILE
}
export -f upload_file
parallel --jobs 50 upload_file {} {/} :::: input_filenames.txt
```

## Copy modified study index
```bash
wget -qO- https://raw.githubusercontent.com/eQTL-Catalogue/eQTL-Catalogue-resources/master/tabix/tabix_ftp_paths_imported.tsv \
  | sed -e 's@ftp://ftp.ebi.ac.uk/pub/databases/spot/eQTL/@gs://genetics_etl_python_playground/input/preprocess/eqtl_catalogue/@g' \
  | gsutil cp - gs://genetics_etl_python_playground/input/preprocess/eqtl_catalogue/tabix_ftp_paths_imported.tsv
```
