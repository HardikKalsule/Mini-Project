#!/bin/bash

# Directory to scan (default is current directory if none provided)
DIR_TO_SCAN=${1:-.}

# Temporary file to store checksums
CHECKSUM_FILE=$(mktemp)

# Find all files in the directory and calculate their MD5 checksums
find "$DIR_TO_SCAN" -type f -exec md5sum {} + | sort > "$CHECKSUM_FILE"

# Extract duplicate files based on checksums
awk '{print $1}' "$CHECKSUM_FILE" | uniq -d > duplicates.txt

# Check if any duplicates were found
if [ -s duplicates.txt ]; then
    echo "Duplicate files found. Removing all but one of each group."
    while read -r checksum; do
        # Get all files with this checksum
        files=($(grep "$checksum" "$CHECKSUM_FILE" | awk '{print $2}'))

        # Print file paths to ensure they're correct
        echo "Files found: ${files[@]}"
        
        # Keep the first file, remove the rest
        echo "Keeping: ${files[0]}"
        for ((i=1; i<${#files[@]}; i++)); do
            # Remove any leading '*' characters from file path
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

# Clean up temporary files
rm "$CHECKSUM_FILE" duplicates.txt
