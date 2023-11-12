# eQTL Catalogue
## Copy summary statistics
```bash
function upload_file () {
  export FILENAME=$1
  SUFFIX=$(echo $FILENAME | sed -e 's@ftp://ftp.ebi.ac.uk/pub/databases/spot/eQTL/@@')
  GS_OUTFILE="gs://genetics_etl_python_playground/input/preprocess/eqtl_catalogue/$SUFFIX"
  wget -qO- $FILENAME | gsutil cp - $GS_OUTFILE
}
export -f upload_file

wget -qO- "https://raw.githubusercontent.com/eQTL-Catalogue/eQTL-Catalogue-resources/master/tabix/tabix_ftp_paths_imported.tsv" \
  | tail -n+2 \
  | cut -f8 \
  | parallel --jobs 50 upload_file
```

## Copy modified study index
```bash
wget -qO- https://raw.githubusercontent.com/eQTL-Catalogue/eQTL-Catalogue-resources/master/tabix/tabix_ftp_paths_imported.tsv \
  | sed -e 's@ftp://ftp.ebi.ac.uk/pub/databases/spot/eQTL/@gs://genetics_etl_python_playground/input/preprocess/eqtl_catalogue/@g' \
  | gsutil cp - gs://genetics_etl_python_playground/input/preprocess/eqtl_catalogue/tabix_ftp_paths_imported.tsv
```
