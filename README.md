# kosmos-documentation

This repository contains the documentation for the use of the internal AI for Oncology cluster Kosmos.

## Installation

To install the necessary requirements for building and viewing the documentation, follow these steps:

1. Clone this repository to your local machine:

   ```shell
   git clone [https://github.com/your-username/kosmos-documentation.git](https://github.com/NKI-AI/kosmos-documentation)
   ```

2. Navigate to the cloned repository:

   ```shell
   cd kosmos-documentation
   ```

3. Install the required dependencies using pip:

   ```shell
   pip install -r requirements.txt
   ```

   This will install the `sphinx_book_theme` package, which is used to handle the rendering of the RST files.

## Building the Documentation

Once the requirements are installed, you can build the documentation using Sphinx. To build the documentation, run the following command:

```shell
make uploaddocs
```

This commands will first clean HTML and other files under the directory `kosmos` if previously `make uploaddocs` was performed. After that it will generate the HTML files for the documentation in the `kosmos` directory.

## Viewing the Documentation
