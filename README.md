# Ansible CSV to HTML Report Generator

This Ansible playbook converts a CSV file into an HTML report, copies both the generated HTML and the source CSV file to a specified directory on a remote server, and provides a download link for the source CSV within the HTML report. It's designed to be easily adaptable for reporting data, such as information extracted from network devices.

## Features

* Converts CSV data to a styled HTML table.
* Uses Ansible's `community.general.read_csv` and Jinja2 templating for conversion.
* Copies the generated HTML and the original CSV file (with a date-prefixed name) to a remote server.
* Inserts a download link in the HTML report for the source CSV file ("right-click to save as" style).
* Prints the URL to access the HTML report on the remote server.
* Includes a sample CSV with data fields typical of Cisco network devices.

## Prerequisites

Before running this playbook, ensure you have the following:

1.  **Ansible:** Installed on your control node (the machine where you run the playbook).
    * Tested with Ansible Core >= 2.12 (likely compatible with newer versions).
2.  **Ansible `community.general` Collection:**
    ```bash
    ansible-galaxy collection install community.general
    ```
3.  **Python:** Ansible itself is Python-based, and the `read_csv` module also relies on it.
4.  **SSH Access:** Configured SSH access (preferably key-based) from the Ansible control node to the target remote server(s).
5.  **Sudo Privileges:** The Ansible user on the remote server may require `sudo` privileges to create the report directory (e.g., `/usr/local/ansible/report`) and copy files into it, depending on the chosen `remote_report_dir`.
6.  **Web Server on Remote Host:** A web server (e.g., Nginx, Apache) must be configured on the remote server to serve files from the `remote_report_dir` so that the generated HTML report is accessible via HTTP/HTTPS.

## Files in the Repository

* **`csv2html-report.yml`**: The main Ansible playbook.
* **`report_template.html.j2`**: The Jinja2 template used to generate the HTML report.
* **`report-example.csv`**: A sample CSV file containing mock data from Cisco network devices. You can replace this with your own CSV file.

## Configuration

You need to customize the following variables in the `csv_to_html_report_templated.yml` playbook:

* **`hosts`**:
    * Default: `all`
    * Change this to the inventory hostname or IP address of your target remote server.
* **`local_csv_file_path`**:
    * Default: `"./report-example.csv"`
    * Path to the source CSV file on your Ansible control node.
* **`report_name`**:
    * Default: `"report"`
    * Base name used for the generated HTML and copied CSV files (e.g., `2025-05-20-report.html`).
* **`remote_report_dir`**:
    * Default: `"/usr/local/ansible/report"`
    * The directory on the remote server where the HTML and CSV files will be copied. Ensure this directory is served by your web server.
* **`http_server_base_url`**:
    * Default: `"http://{{ ansible_host }}"`
    * The base URL of your web server on the remote host (e.g., `http://192.168.1.100` or `http://server.example.com`).
    * **Important**: The final URL displayed by the playbook is constructed as `{{ http_server_base_url }}{{ remote_report_dir | urlencode }}/{{ html_file_name }}`. If your web server maps `remote_report_dir` to a different URL path (e.g., `/reports/` instead of `/usr/local/ansible/report/`), you will need to adjust the `msg` line in the last task ("Display URL to access the HTML report") accordingly. For example: `msg: "Report available at: {{ http_server_base_url }}/reports/{{ html_file_name }}"`

## How to Use

1.  **Clone the Repository (or download the files):**
    ```bash
    # git clone <repository_url>
    # cd <repository_directory>
    ```
2.  **Install Ansible Collection (if not already done):**
    ```bash
    ansible-galaxy collection install community.general
    ```
3.  **Prepare your CSV data:**
    * You can use the provided `report-example.csv` for testing.
    * To use your own data, replace `report-example.csv` or update the `local_csv_file_path` variable in the playbook.
4.  **Configure the Playbook:**
    * Open `csv2html-report.yml` and modify the variables in the `vars` section as described under "Configuration."
5.  **Run the Playbook:**
    ```bash
    ansible-playbook csv_to_html_report_templated.yml
    ```
    If you don't use a pre-configured inventory file or need to specify connection details:
    ```bash
    ansible-playbook csv_to_html_report_templated.yml -i your_remote_server_ip, --user your_ssh_user --ask-become-pass
    ```
    (Replace `your_remote_server_ip` and `your_ssh_user` accordingly).

## Expected Output

* An HTML file (e.g., `YYYY-MM-DD-report.html`) and a CSV file (e.g., `YYYY-MM-DD-report.csv`) will be created in the `remote_report_dir` on the target server.
* The console will display a message with the URL to access the HTML report, similar to:
    `Report available at: http://<remote_server_ip_or_hostname>/usr/local/ansible/report/YYYY-MM-DD-report.html`
* Opening this URL in a web browser will show the CSV data rendered as an HTML table.
* At the bottom of the HTML page, there will be a link to download the source CSV file.

## Customizing the HTML Output

The appearance of the HTML report (styling, layout) can be modified by editing the `report_template.html.j2` Jinja2 template file. You can change the CSS styles, table structure, or add more information as needed.

---

*Generated: May 20, 2025*