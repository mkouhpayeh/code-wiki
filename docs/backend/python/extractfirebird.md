# Extract Firebird .sql data


``` py
import os
import re
import binascii
import zlib

# 📑 File paths
sql_file_path = r"Export_Documents.sql"
output_dir = r"output_files"
os.makedirs(output_dir, exist_ok=True)

# 📑 Regex to match each VALUES row: ('AMSIDNR','BFTYP',0xHEXDATA)
pattern = re.compile(
    r"\(\s*'(?P<amsidnr>[^']+)',\s*'(?P<bftyp>[^']+)',\s*0x(?P<hexdata>[0-9A-F]+)\s*\)"
)

file_counter = 0
parsing_started = False

def process_line(line):
    global file_counter
    matches = pattern.finditer(line)
    for match in matches:
        raw_amsidnr = match.group("amsidnr")

	# 📑 Remove leading zeros
        amsidnr = raw_amsidnr.lstrip("0")  
        bftyp = match.group("bftyp")
        hexdata = match.group("hexdata")

        # 📑 Clean extension:
        extension = bftyp.lower()

        # 📑 Build filename
        filename = f"{amsidnr}.{extension}"
        filepath = os.path.join(output_dir, filename)

        try:
            compressed_data = binascii.unhexlify(hexdata)

            # 📑 Starts with '0x789C' => zlib-compressed
            decompressed_data = zlib.decompress(compressed_data)

            with open(filepath, "wb") as f:
                f.write(decompressed_data)

            file_counter += 1
            if file_counter % 1000 == 0:
                print(f"✅ Processed {file_counter} files...")
        except Exception as e:
            print(f"❌ Error saving {filepath}: {e}")

def process_sql_file(file_path):
    global parsing_started
    with open(file_path, "r", encoding="utf-8") as file:
        buffer = ""
        for line in file:
            line = line.strip()
            if not parsing_started:
                if "VALUES" in line:
                    parsing_started = True
                    print("🟢 VALUES section found, starting extraction...")
                    buffer = ""
                continue

            if parsing_started:
                buffer += line
                if buffer.endswith(");") or line.endswith("),"):
                    process_line(buffer)
                    buffer = ""

# 📑 Run the extraction
process_sql_file(sql_file_path)
print(f"🎉 All done! {file_counter} files extracted and saved to: {output_dir}")

```
