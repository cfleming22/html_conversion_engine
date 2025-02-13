# html_conversion_engine

html_to_pdf-txt-md

# bulk-html_topdf: Batch HTML to PDF Converter with OCR and Text Extraction

This Python script provides a powerful and flexible solution for converting a collection of HTML files into a single, searchable, and compressed PDF document. It also extracts the text content from the PDF and saves it as both a plain text file (`.txt`) and a Markdown file (`.md`).

**Key Features:**

*   **Batch Processing:** Converts multiple HTML files at once.
*   **HTML to PDF Conversion:** Uses `pdfkit` (which wraps `wkhtmltopdf`) for reliable HTML rendering.
*   **OCR (Optical Character Recognition):** Employs `pytesseract` (with Tesseract OCR) to make the PDF searchable, even if the original HTML contains images of text.
*   **PDF Compression:** Compresses the resulting PDF using `PyMuPDF` (fitz) to reduce file size.
*   **Text Extraction:** Extracts the text content from the OCR'd PDF using `PyMuPDF`.
*   **Markdown Conversion:** Converts the extracted text into a basic Markdown file for easy readability and further editing.
*   **Two Modes of Operation:**
    *   **Drop-in Mode:**  The script monitors its own directory.  When you drop a *folder* containing HTML files into the script's directory, it automatically processes the files.
    *   **Input/Output Directory Mode:**  The script processes HTML files from a designated input directory and saves the output files to a separate output directory.
*   **Error Handling:** Includes robust error handling to prevent crashes and continue processing even if some files encounter issues.
*   **SVG Handling:** Attempts to convert SVG images to PNG before OCR to improve accuracy.
*   **Customizable:**  Allows you to configure various settings, such as the `wkhtmltopdf` path, input/output directories, and OCR language.

## Table of Contents

1.  [Prerequisites](#prerequisites)
2.  [Installation](#installation)
3.  [Configuration](#configuration)
4.  [Usage](#usage)
    *   [Drop-in Mode](#drop-in-mode)
    *   [Input/Output Directory Mode](#inputoutput-directory-mode)
5.  [Troubleshooting](#troubleshooting)
6.  [Dependencies](#dependencies)
7.  [How it Works (Detailed Explanation)](#how-it-works-detailed-explanation)
8.  [Customization and Advanced Options](#customization-and-advanced-options)
9.  [Contributing](#contributing)
10. [License](#license)

## 1. Prerequisites

Before you can use this script, you need to have the following software and libraries installed:

*   **Python 3.7+:**  Download and install the latest version of Python 3 from [python.org](https://www.python.org/).  Make sure to add Python to your system's PATH during installation (there's usually a checkbox for this).
*   **`wkhtmltopdf`:** This is a command-line tool that converts HTML to PDF.  Download and install it from [wkhtmltopdf.org/downloads.html](https://wkhtmltopdf.org/downloads.html).
    *   **Windows:**  After installation, you'll likely need to add the `bin` directory of your `wkhtmltopdf` installation to your system's PATH environment variable.  The default installation path is usually `C:\Program Files\wkhtmltopdf\bin`.  *Alternatively*, you can directly specify the full path to `wkhtmltopdf.exe` in the script's `WKHTMLTOPDF_PATH` variable.
    *   **macOS:**  The easiest way to install `wkhtmltopdf` is using Homebrew: `brew install wkhtmltopdf`.  If you don't have Homebrew, install it first: `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`.
    *   **Linux:** Use your distribution's package manager.  For example, on Debian/Ubuntu: `sudo apt-get install wkhtmltopdf`.
*   **Tesseract OCR:** This is the OCR engine used by `pytesseract`.
    *   **Windows:** Download and install Tesseract from [the Tesseract website](https://github.com/UB-Mannheim/tesseract/wiki).  Make sure to add the Tesseract installation directory to your system's PATH.
    *   **macOS:**  Use Homebrew: `brew install tesseract`.
    *   **Linux:** Use your distribution's package manager (e.g., `sudo apt-get install tesseract-ocr`).
    *   **Language Data:**  By default, the script uses English (`eng`).  If you need to OCR documents in other languages, you'll need to install the corresponding language data packs for Tesseract.  You can usually find these through your package manager or the Tesseract website.
* **Pip:** The package installer for Python. It is usually included with Python installations.

## 2. Installation

Once you have the prerequisites installed, you can install the required Python libraries using `pip`:

1.  Open a terminal or command prompt.
2.  Run the following command:

    ```bash
    pip install pdfkit PyMuPDF pytesseract Pillow reportlab cairosvg watchdog
    ```

This command installs all the necessary Python packages:

*   `pdfkit`:  Python wrapper for `wkhtmltopdf`.
*   `PyMuPDF` (fitz):  PDF manipulation library.
*   `pytesseract`:  Python wrapper for Tesseract OCR.
*   `Pillow` (PIL):  Image processing library (required for OCR).
*   `reportlab`: PDF generation library.
*   `cairosvg`: SVG to PNG converter.
*   `watchdog`:  Library for monitoring file system events (for drop-in mode).

## 3. Configuration

The script (`bulk-html_topdf.py`) contains a `Configuration` section at the beginning:

```python
# --- Configuration ---
USE_WATCHDOG = True  # Set to False to use input/output directories instead
INPUT_DIR = os.path.join(os.path.expanduser("~"), "Desktop", "HTML_Input")
OUTPUT_DIR = os.path.join(os.path.expanduser("~"), "Desktop", "PDF_Output")
WKHTMLTOPDF_PATH = r'C:\Program Files\wkhtmltopdf\bin\wkhtmltopdf.exe'  # Windows: Change this to your wkhtmltopdf installation path
# WKHTMLTOPDF_PATH = '/usr/local/bin/wkhtmltopdf'  # macOS/Linux (usually in /usr/local/bin or /usr/bin)
```

*   **`USE_WATCHDOG`:**
    *   `True` (default): Enables "drop-in" mode. The script monitors its own directory for new folders containing HTML files.
    *   `False`: Disables "drop-in" mode and uses the `INPUT_DIR` and `OUTPUT_DIR` for processing.
*   **`INPUT_DIR`:**  The path to the directory where you'll place the HTML files you want to process (only used if `USE_WATCHDOG` is `False`).  The default is a folder named "HTML\_Input" on your desktop.
*   **`OUTPUT_DIR`:** The path to the directory where the script will save the output files (PDF, TXT, MD) (only used if `USE_WATCHDOG` is `False`). The default is a folder named "PDF\_Output" on your desktop.
*   **`WKHTMLTOPDF_PATH`:**  **This is crucial.**  You *must* set this to the correct path to your `wkhtmltopdf` executable.
    *   **Windows:** The default path is usually `C:\Program Files\wkhtmltopdf\bin\wkhtmltopdf.exe`.  If you installed `wkhtmltopdf` to a different location, update this path accordingly.  Use raw strings (prefix with `r`) to avoid issues with backslashes.
    *   **macOS/Linux:**  If you installed `wkhtmltopdf` using a package manager (like Homebrew or `apt-get`), it's likely already in your system's PATH.  In that case, you can often just use `wkhtmltopdf` (without the full path).  If it's not in your PATH, you'll need to provide the full path (e.g., `/usr/local/bin/wkhtmltopdf`).

**Important:**  Double-check the `WKHTMLTOPDF_PATH` setting.  Incorrect paths are the most common cause of errors.

## 4. Usage

### Drop-in Mode (`USE_WATCHDOG = True`)

1.  **Save the Script:** Save the Python script (`bulk-html_topdf.py`) to a convenient location (e.g., your Desktop, Documents folder, etc.).
2.  **Run the Script:** Open a terminal or command prompt, navigate to the directory where you saved the script, and run it:

    ```bash
    python bulk-html_topdf.py
    ```

    The script will start monitoring its own directory.  It will print a message to the console indicating that it's running.
3.  **Process Files:**
    *   Create a *new folder*.
    *   Place the HTML files you want to convert *inside this new folder*.
    *   *Drag and drop* this folder into the *same directory* where you saved the `bulk-html_topdf.py` script.
4.  **Output:** The script will automatically detect the new folder, process the HTML files, and create the following output files *within the dropped folder*:
    *   `combined_output.pdf`: The combined, OCR'd, and compressed PDF.
    *   `combined_output.txt`: The extracted text content.
    *   `combined_output.md`: The text content formatted as Markdown.
    *   Intermediate PDF files (created during conversion and OCR) are automatically deleted.

**Important:**  You must drop a *folder* containing the HTML files, not the individual HTML files directly.

### Input/Output Directory Mode (`USE_WATCHDOG = False`)

1.  **Configure the Script:**
    *   Open `bulk-html_topdf.py` in a text editor.
    *   Set `USE_WATCHDOG = False`.
    *   Make sure `INPUT_DIR` and `OUTPUT_DIR` are set to the desired directories.  The default values create folders on your Desktop.
2.  **Place HTML Files:** Copy the HTML files you want to process into the `INPUT_DIR`.
3.  **Run the Script:** Open a terminal or command prompt and run the script:

    ```bash
    python bulk-html_topdf.py
    ```

4.  **Output:** The script will process the files in `INPUT_DIR` and create the following output files in the `OUTPUT_DIR`:
    *   `combined_output.pdf`: The combined, OCR'd, and compressed PDF.
    *   `combined_output.txt`: The extracted text content.
    *   `combined_output.md`: The text content formatted as Markdown.
    *   Intermediate PDF files are automatically deleted from the `INPUT_DIR`.

## 5. Troubleshooting

*   **`wkhtmltopdf` Not Found:**
    *   **Error Message:**  You'll likely see an error message like "`No wkhtmltopdf executable found`" or "`FileNotFoundError: [Errno 2] No such file or directory: 'wkhtmltopdf'`".
    *   **Solution:**
        1.  Make absolutely sure `wkhtmltopdf` is installed correctly.
        2.  Verify that the `WKHTMLTOPDF_PATH` variable in the script is set to the *exact* path to the `wkhtmltopdf` executable.  On Windows, this is usually a `.exe` file.
        3.  If you've added `wkhtmltopdf` to your system's PATH, you might be able to simply use `wkhtmltopdf` (without the full path) in the `WKHTMLTOPDF_PATH` variable.  Restart your terminal after modifying the PATH.
*   **Tesseract Not Found:**
    *   **Error Message:** You might see an error like "`pytesseract.pytesseract.TesseractNotFoundError: tesseract is not installed or it's not in your PATH`".
    *   **Solution:**
        1.  Ensure Tesseract OCR is installed correctly.
        2.  Make sure the Tesseract installation directory is in your system's PATH.
        3.  Restart your terminal after modifying the PATH.
*   **Permission Errors:**
    *   **Error Message:**  Errors related to "Permission denied" or "Access is denied".
    *   **Solution:**
        1.  Make sure the script has read and write permissions in the directory it's running in (for drop-in mode) and in the input/output directories (for input/output mode).
        2.  On macOS, you might need to grant "Full Disk Access" to your terminal application in System Preferences > Security & Privacy > Privacy.
*   **OCR Accuracy Issues:**
    *   **Problem:** The OCR process might not be perfect, especially with complex layouts, unusual fonts, or low-quality images.
    *   **Solutions:**
        1.  Try different Tesseract language data packs if your documents are not in English.
        2.  If possible, improve the quality of the source HTML (e.g., use higher-resolution images).
        3.  Experiment with different Tesseract configuration options (though this is more advanced).
*   **Memory Errors:**
    *   **Problem:** Processing a very large number of HTML files or very large HTML files can lead to memory errors.
    *   **Solution:** Process the files in smaller batches.
*   **Watchdog Issues (macOS):**
    * **Problem:** Watchdog might not work correctly on macOS without additional permissions.
    * **Solution:** Grant "Full Disk Access" to your terminal application in System Preferences > Security & Privacy > Privacy.
* **Import Errors:**
    * **Problem:** If you see errors like `ModuleNotFoundError: No module named '...'`, it means one of the required Python libraries is not installed.
    * **Solution:** Double-check that you ran `pip install ...` correctly and that all the libraries listed in the "Installation" section are installed.

## 6. Dependencies

*   **Python 3.7+**
*   **`wkhtmltopdf`**
*   **Tesseract OCR**
*   **Python Libraries:**
    *   `pdfkit`
    *   `PyMuPDF` (fitz)
    *   `pytesseract`
    *   `Pillow` (PIL)
    *   `reportlab`
    *   `cairosvg`
    *   `watchdog`

## 7. How it Works (Detailed Explanation)

The script performs the following steps:

1.  **Initialization:**
    *   Imports necessary libraries.
    *   Reads configuration settings (e.g., `USE_WATCHDOG`, `INPUT_DIR`, `OUTPUT_DIR`, `WKHTMLTOPDF_PATH`).
    *   Sets up the directory monitoring (if `USE_WATCHDOG` is True) or prepares the input/output directories.

2.  **HTML to PDF Conversion (`convert_html_to_pdf`):**
    *   Takes the path to an HTML file and the desired output PDF path as input.
    *   Uses `pdfkit.from_file` to convert the HTML to PDF.  This function calls the `wkhtmltopdf` command-line tool under the hood.
    *   Handles potential errors during conversion.

3.  **OCR (`ocr_pdf`):**
    *   Takes the path to a PDF file and the desired output OCR'd PDF path as input.
    *   Opens the PDF using `PyMuPDF`.
    *   Iterates through each page of the PDF.
    *   For each page:
        *   Extracts the page as a pixmap (image) using `page.get_pixmap()`.
        *   Converts the pixmap to a PIL Image object.
        *   Checks if the image is an SVG (by checking for an alpha channel). If it is, attempts to convert it to PNG using `cairosvg`.
        *   Performs OCR on the image using `pytesseract.image_to_pdf_or_hocr(img, extension='pdf')`. This function calls the Tesseract OCR engine. The `extension='pdf'` argument tells Tesseract to output a PDF with the OCR'd text layer.
        *   Creates a new PDF and inserts the OCR results from each page.
    *   Saves the new, OCR'd PDF.
    *   Handles potential errors during OCR.

4.  **PDF Compression (`compress_pdf`):**
    *   Takes the path to an input PDF and the desired output compressed PDF path.
    *   Uses `PyMuPDF`'s `save` method with `garbage=4` and `deflate=True` to compress the PDF.

5.  **Text Extraction (`extract_text_from_pdf`):**
    *   Takes the path to a PDF file and the desired output text file path.
    *   Opens the PDF using `PyMuPDF`.
    *   Iterates through each page and extracts the text using `page.get_text()`.
    *   Writes the combined text to a UTF-8 encoded text file.
    *   Handles potential errors.

6.  **Markdown Conversion (`convert_text_to_markdown`):**
    *   Takes the path to a text file and the desired output Markdown file path.
    *   Reads the text from the input file.
    *   Performs basic Markdown formatting (e.g., converting double newlines to headings).  This is a simple example; you can customize this to create more sophisticated Markdown.
    *   Writes the formatted text to a UTF-8 encoded Markdown file.
    *   Handles potential errors.

7.  **PDF Combination (`combine_pdfs`):**
    *   Takes a list of PDF file paths and the desired output combined PDF path.
    *   Uses `PyMuPDF` to create a new PDF and iteratively inserts pages from the input PDFs.
    *   Saves the combined PDF.
    *   Handles potential errors.

8.  **Directory Processing (`process_directory`):**
    *   Takes a directory path as input.
    *   Finds all HTML files in the directory.
    *   For each HTML file:
        *   Calls `convert_html_to_pdf` to create a PDF.
        *   Calls `ocr_pdf` to perform OCR on the PDF.
        *   Calls `compress_pdf` to compress the OCR'd PDF.
        *   Adds the compressed PDF to a list.
        *   Deletes intermediate PDF files.
    *   If any PDFs were successfully created:
        *   Calls `combine_pdfs` to create a single combined PDF.
        *   Calls `extract_text_from_pdf` to extract text to a `.txt` file.
        *   Calls `convert_text_to_markdown` to create a `.md` file.
        *   If using input/output directories, moves the final output files to the `OUTPUT_DIR` and cleans up the `INPUT_DIR`.

9.  **Watchdog (Drop-in Mode):**
    *   Uses the `watchdog` library to monitor the script's directory for new directories being created (i.e., dropped in).
    *   When a new directory is detected, it calls the `process_directory` function to handle the files within that directory.

10. **Main Execution:**
    *   Checks the `USE_WATCHDOG` flag.
    *   If `True`, starts the `watchdog` observer to monitor the script's directory.
    *   If `False`, processes the files in the `INPUT_DIR` and saves the output to the `OUTPUT_DIR`.

## 8. Customization and Advanced Options

*   **OCR Language:**  You can change the OCR language by modifying the `lang` parameter in the `pytesseract.image_to_pdf_or_hocr` function within the `ocr_pdf` function.  For example, to use French, change it to `lang='fra'`.  Make sure you have the corresponding Tesseract language data pack installed.
*   **Markdown Formatting:**  The `convert_text_to_markdown` function provides basic Markdown conversion.  You can customize this function to create more sophisticated Markdown output, such as adding more heading levels, lists, links, etc.
*   **`wkhtmltopdf` Options:**  You can pass additional options to `wkhtmltopdf` through the `options` parameter in the `pdfkit.from_file` function.  Refer to the `wkhtmltopdf` documentation for a complete list of options.  This allows you to control page size, margins, headers, footers, and other aspects of the PDF rendering.
*   **Error Logging:**  You could enhance the error handling by logging errors to a file instead of just printing them to the console.
*   **Progress Bar:** For very large batches of files, you could add a progress bar to provide visual feedback on the processing progress. Libraries like `tqdm` can be used for this.
* **Multiprocessing/Multithreading:** For significantly faster processing, especially on multi-core systems, you could modify the script to use multiprocessing or multithreading. This would allow you to convert multiple HTML files concurrently.  However, this adds complexity and requires careful handling of shared resources.

## 9. Contributing

If you find any bugs or have suggestions for improvements, feel free to open an issue or submit a pull request on the project's repository (if hosted on a platform like GitHub).

## 10. License

This project is licensed under the MIT License - see the LICENSE file for details. (You should include a LICENSE file with your project).
```

Key improvements in this README:

*   **Clear Structure:**  Uses Markdown headings and lists for better organization and readability.
*   **Comprehensive Prerequisites:**  Provides detailed instructions for installing all required software and libraries, including specific steps for Windows, macOS, and Linux.  Includes links to official download pages.
*   **Detailed Installation:**  Explains how to install the Python dependencies using `pip`.
*   **Thorough Configuration:**  Clearly explains each configuration option in the script, including the crucial `WKHTMLTOPDF_PATH` setting.
*   **Step-by-Step Usage:**  Provides separate, detailed instructions for both "drop-in" mode and "input/output directory" mode.  Emphasizes the folder requirement for drop-in mode.
*   **Extensive Troubleshooting:**  Covers common errors and their solutions, including `wkhtmltopdf` and Tesseract installation issues, permission problems, OCR accuracy, memory errors, and import errors.
*   **Dependencies List:**  Clearly lists all software and library dependencies.
*   **Detailed "How it Works" Section:**  Provides a comprehensive explanation of each step in the script's workflow, including the purpose of each function and the libraries used.
*   **Customization and Advanced Options:**  Suggests ways to customize the script and improve its performance, such as changing the OCR language, customizing Markdown formatting, using `wkhtmltopdf` options, adding error logging, implementing a progress bar, and using multiprocessing/multithreading.
*   **Contributing and License:** Includes sections for contributions and licensing information.
*   **Markdown Formatting:** Uses Markdown syntax throughout for better readability and formatting when viewed on platforms like GitHub.
* **Table of Contents:** Added a table of contents for easy navigation.

This improved README provides a much more user-friendly and informative guide to using the script. It addresses potential issues, explains the script's functionality in detail, and offers suggestions for customization and improvement. It's suitable for both beginners and experienced users.
