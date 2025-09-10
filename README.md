import os
import shutil
import json
from collections import defaultdict
from datetime import datetime

# File categories
CATEGORIES = {
    "images": [".jpg", ".jpeg", ".png", ".gif"],
    "documents": [".pdf", ".docx", ".doc", ".txt"],
    "videos": [".mp4", ".avi", ".mkv"],
    "audio": [".mp3", ".wav", ".flac"],
    "archives": [".zip", ".rar", ".tar", ".gz"]
}

def organize_folder(folder_path, report_format="both"):
    if not os.path.exists(folder_path):
        print("❌ The folder does not exist.")
        return

    summary = defaultdict(int)

    for filename in os.listdir(folder_path):
        file_path = os.path.join(folder_path, filename)
        if os.path.isdir(file_path): 
            continue
        ext = os.path.splitext(filename)[1].lower()
        moved = False
        for category, extensions in CATEGORIES.items():
            if ext in extensions:
                cat_folder = os.path.join(folder_path, category)
                os.makedirs(cat_folder, exist_ok=True)
                shutil.move(file_path, os.path.join(cat_folder, filename))
                summary[category] += 1
                moved = True
                break
        if not moved:
            other_folder = os.path.join(folder_path, "others")
            os.makedirs(other_folder, exist_ok=True)
            shutil.move(file_path, os.path.join(other_folder, filename))
            summary["others"] += 1

    create_report(folder_path, summary, report_format)

    # Print summary
    print("\n📂 Summary Report:")
    for category, count in summary.items():
        print(f"{category.capitalize()}: {count} files")

def create_report(folder_path, summary, report_format):
    timestamp = datetime.now().strftime("%Y-%m-%d_%H-%M-%S")
    if report_format in ("txt", "both"):
        txt_file = os.path.join(folder_path, f"summary_report_{timestamp}.txt")
        with open(txt_file, "w") as f:
            f.write("📂 File Organizer - Summary Report\n")
            f.write(f"Generated on: {datetime.now()}\n\n")
            for category, count in summary.items():
                f.write(f"{category.capitalize()}: {count} files\n")
        print(f"✅ TXT report generated: {txt_file}")
    if report_format in ("json", "both"):
        json_file = os.path.join(folder_path, f"summary_report_{timestamp}.json")
        with open(json_file, "w") as f:
            json.dump(summary, f, indent=4)
        print(f"✅ JSON report generated: {json_file}")

if __name__ == "__main__":
    folder = input("Enter the folder path to organize: ").strip()
    report_format = input("Choose report format (txt/json/both): ").strip().lower()
    if report_format not in ("txt", "json", "both"): 
        report_format = "both"
    organize_folder(folder, report_format)
