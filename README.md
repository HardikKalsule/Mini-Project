# Mini-Project
#!/bin/bash
DIR_TO_SCAN=${1:-.}
CHECKSUM_FILE=$(mktemp)
find "$DIR_TO_SCAN" -type f -exec md5sum {} + | sort > "$CHECKSUM_FILE"
awk '{print $1}' "$CHECKSUM_FILE" | uniq -d > duplicates.txt
if [ -s duplicates.txt ]; then
    echo "Duplicate files found. Removing all but one of each group."
    while read -r checksum; do
        files=($(grep "$checksum" "$CHECKSUM_FILE" | awk '{print $2}'))
        echo "Files found: ${files[@]}"
        echo "Keeping: ${files[0]}"
        for ((i=1; i<${#files[@]}; i++)); do
            clean_path=$(echo "${files[$i]}" | sed 's/^\*//')
            echo "Attempting to remove: $clean_path"
            if [ -f "$clean_path" ]; then
                echo "Removing: $clean_path"
                rm -f "$clean_path"
            else
                echo "File not found: $clean_path"
            fi
        done
    done < duplicates.txt
else
    echo "No duplicate files found."
fi
rm "$CHECKSUM_FILE" duplicates.txt
