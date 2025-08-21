## A Guide to Converting .docx to .rst Files with a Modified `rstdoc` Workflow

This guide walks you through setting up a custom workflow to convert Microsoft Word (`.docx`) files into reStructuredText (`.rst`) files. You'll start by installing the necessary tools, then fork and modify the `rstdoc` library to suit specific needs, and finally, install and use your custom version.

### Prerequisites: Initial Software Setup

Before you begin, you need to install two key pieces of software: **Pandoc** (a universal document converter) and **pypandoc** (a Python wrapper for Pandoc).

1.  **Install Pandoc:**
    * Download and install the appropriate version for your operating system from the official release page: [Pandoc 3.6.2 Releases](https://github.com/jgm/pandoc/releases/tag/3.6.2).
    * After installation, open your terminal or command prompt and run the following command to verify it was installed correctly:
        ```bash
        pandoc --version
        ```

2.  **Install pypandoc:**
    * Run the following command in your terminal:
        ```bash
        pip install pypandoc
        ```

### Part 1: Creating Your Custom `rstdoc` Library

The core of this workflow involves creating a personalized version of the `rstdoc` library.

#### **Step 1: Fork and Clone the Repository**

1.  Navigate to the [rstdoc GitHub repository](https://github.com/rstdoc/rstdoc).

2.  Click the **Fork** button in the top-right corner to create a personal copy of the repository under your GitHub account.

3.  Name your forked repository (e.g., `rstdoc-xgrid-workflow`) and add a brief description (e.g., 
*This is a tool to support RST documentation Word-to-RST conversion using Pandoc*).

4.  On your local machine, create a project folder (e.g., `rstdoc-project`) and navigate into it.

5.  Clone your forked repository into this folder. Replace the URL with your repository's URL.
    ```bash
    git clone [https://github.com/YourUsername/rstdoc-xgrid-workflow.git](https://github.com/YourUsername/rstdoc-xgrid-workflow.git)
    ```

6.  Create folders to hold your source documents and the final output files.
    ```bash
    mkdir source_docs
    mkdir final_rst
    ```

#### **Step 2: Modify the Python Files**

Navigate into your cloned repository's `rstdoc` subfolder (e.g., `rstdoc-xgrid-workflow/rstdoc/`) and make the following code changes.

1.  **Change File Extension to `.rst`:**
    * Open **`fromdocx.py`**.
    * Locate the `_docxrest(adocx)` function.
    * Change all instances of the `.rest` file extension to `.rst`.

2.  **Adjust Configuration:**
    * Open **`dcx.py`**.
    * Modify the `conf` settings as needed, ensuring you maintain uniform indentation.

3.  **Prevent Unwanted File Copying:**
    * The `rstfromdocx` function copies the `dcx.py` file by default. To prevent this, open **`fromdocx.py`** and comment out the call to the `write_dcx` function within the `main` function.

4.  **Fix Image Paths (Backslash Issue):**
    * To ensure image paths use forward slashes (`/`) instead of backslashes (`\`), open **`reimg.py`**.
    * Add the `posixpath` import at the top of the file:
        ```python
        import posixpath
        ```
    * Find the line responsible for joining paths and replace it with the following:
        ```python
        newp = posixpath.join(d, newn)
        ```

5.  **Add Support for Admonition Directives:**
    * Pandoc doesn't automatically convert admonitions like "Note:" or "Warning:" into proper RST directives. To fix this, open **`fromdocx.py`** and add the following helper function anywhere in the file.

    ```python
    ## FOLLOWING FUNCTION CONVERTS NOTES ETC INTO THEIR RESPECTIVE DIRECTIVE
    import re

    def process_rst_admonitions(rst_text):
        """
        Correctly converts multi-line admonitions (e.g., "**Note**: ...") into
        properly indented RST directives. This version is robust and handles
        keywords that may be formatted with bold or italics by Pandoc.

        Args:
            rst_text (str): The input RST text generated from Pandoc.

        Returns:
            str: The processed RST text with correctly formatted multi-line directives.
        """
        directive_mappings = {
            "note": "note", "see also": "seealso", "attention": "attention",
            "caution": "caution", "error": "error", "danger": "danger",
            "hint": "hint", "tip": "tip", "important": "important", "warning": "warning",
        }
        keywords_pattern = "|".join(re.escape(key) for key in directive_mappings.keys())
        pattern = re.compile(
            rf"^\s*[\*_]*\s*({keywords_pattern})\s*[\*_]*\s*:\s*(.*)",
            re.IGNORECASE
        )
        paragraphs = re.split(r'\n\s*\n', rst_text)
        processed_paragraphs = []
        for paragraph in paragraphs:
            if not paragraph.strip():
                processed_paragraphs.append(paragraph)
                continue
            lines = paragraph.split('\n')
            first_line = lines[0].strip()
            match = pattern.match(first_line)
            if match:
                keyword = match.group(1).lower().strip()
                rest_of_first_line = match.group(2).strip()
                directive = directive_mappings.get(keyword, "note")
                if rest_of_first_line:
                    new_lines = [f".. {directive}:: {rest_of_first_line}"]
                else:
                    new_lines = [f".. {directive}::"]
                for line in lines[1:]:
                    if line.strip():
                        new_lines.append(f"   {line.rstrip()}")
                    else:
                        new_lines.append("   ")
                processed_paragraphs.append('\n'.join(new_lines))
            else:
                processed_paragraphs.append(paragraph)
        return '\n\n'.join(processed_paragraphs)
    ```

6.  **Update the `main` Function:**
    * Next, replace the existing `main` function in **`fromdocx.py`** with this modified version. This new version integrates the `process_rst_admonitions` function into the conversion pipeline.

    ```python
    def main(**args):
        '''
        This corresponds to the |rstfromdocx| shell command.
        '''
        import argparse
        if not args:
            parser = argparse.ArgumentParser(
                description='''Convert DOCX to RST using Pandoc and additionally copy the images and helper files.'''
            )
            parser.add_argument('--version', action='version', version = __version__)
            parser.add_argument('docx', action='store', help='DOCX file')
            parser.add_argument('-l', '--listtable', action='store_true', default=False, help='''postprocess through rstlisttable''')
            parser.add_argument('-u', '--untable', action='store_true', default=False, help='''postprocess through rstuntable''')
            parser.add_argument('-r', '--reflow', action='store_true', default=False, help='''postprocess through rstreflow''')
            parser.add_argument('-g', '--reimg', action='store_true', default=False, help='''postprocess through rstreimg''')
            parser.add_argument('-j', '--join', action='store', default='012', help='''e.g.002. Join method per column: 0="".join; 1=" ".join; 2="\\n".join''')
            args = parser.parse_args().__dict__

        adocx = args['docx']
        extract_media(adocx)
        fnrst = _docxrst(adocx)
        
        # Perform the standard Pandoc conversion
        rst = pypandoc.convert_file(adocx, 'rst', 'docx')

        # Process admonitions (notes, warnings, etc.) into proper RST directives
        processed_rst = process_rst_admonitions(rst)

        # Write the processed content to the RST file
        with open(fnrst, 'w', encoding='utf-8', newline='\n') as f:
            f.write('.. vim: syntax=rst\n\n')
            f.write(processed_rst)
            
        _write_confpy(adocx)
        _write_index(adocx)
        _write_makefile(adocx)

        # Set default values for post-processing options
        default_options = {'listtable': False, 'untable': False, 'reflow': False, 'reimg': False, 'join': '012'}
        
        # Update args with defaults if not provided
        for key, default_value in default_options.items():
            if key not in args:
                args[key] = default_value

        # Apply post-processing options
        for processor in ['listtable', 'untable', 'reflow', 'reimg']:
            if args[processor]:
                args['in_place'] = True
                args['sentence'] = True
                if processor == 'reflow':
                    args['join'] = '0'
                args['rstfile'] = [argparse.FileType('r', encoding='utf-8')(fnrst)]
                eval(processor)(**args)
        return fnrst
    ```

7.  **(Optional) Add Multiple File Support:**
    * To process multiple Word files, you would need to further modify the functions responsible for creating configuration and index files (`extract_media`, `_write_confpy`, `_write_index`, `_write_makefile`) and add logic to split documents or iterate through multiple files. This custom logic should be added here.

---

### Part 2: Installing and Using Your Custom Library

Now that you've modified the code, you can install it and use it.

#### **Step 1: Install the Modified Library**

* Install your local, modified version of `rstdoc` using `pip`'s editable mode (`-e`). This links the installation to your source files, so any further changes you make are immediately reflected.
    ```bash
    pip install -e /path/to/your/rstdoc-xgrid-workflow
    ```

#### **Step 2: Run the Conversion Command**

* You can now convert a `.docx` file using the `rstfromdocx` command. The `-lurg` flags apply all common post-processing steps.
    ```bash
    rstfromdocx -lurg your_document.docx
    ```
    This command performs the following actions:
    * Converts the `.docx` file to an `.rst` file inside a new subfolder.
    * Creates helper files like `conf.py`, `index.py`, and `Makefile`.
    * Applies post-processing options:
        * `--listtable` or `-l`: Converts lists into tables.
        * `--untable` or `-u`: Converts tables into plain text.
        * `--reflow` or `-r`: Reflows paragraphs to fix awkward line breaks.
        * `--reimg` or `-g`: Ensures image references are correctly formatted.

### Part 3: Troubleshooting

#### **Fixing "command not found" on macOS**

If your terminal shows a `command not found` error for `rstfromdocx`, it means the installation location is not in your shell's `PATH`.

1.  **Find the script directory.** For Python 3.9, this is typically `~/Library/Python/3.9/bin`.

2.  **Edit your Z shell configuration file** using a text editor like `nano`.
    ```bash
    nano ~/.zshrc
    ```
3.  **Add the path.** Go to the end of the file and add the following line, adjusting the path if necessary:
    ```bash
    export PATH="$HOME/Library/Python/3.9/bin:$PATH"
    ```
4.  **Save and exit.** Press `Control + O` to save, then `Control + X` to exit.

5.  **Reload your terminal configuration.** You can either close and reopen the terminal or run:
    ```bash
    source ~/.zshrc
    ```

6.  **Verify the command.** Check that the shell can now find the command:
    ```bash
    which rstfromdocx
    ```
    This should print the full path to the command, and you can now run your conversions successfully.