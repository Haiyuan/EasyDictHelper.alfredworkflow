# URL Converter Service Setup Guide

This project sets up a local HTTP server that listens for requests of the form `http://localhost:8080/?text={word}` and converts them into `easydict://query?text={word}`, which is then opened by the default web browser. The service is configured to run at startup on MacOS.

## Prerequisites

1. **Homebrew**: If you don't have Homebrew installed, you can install it with the following command:
   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **Python**: Install Python via Homebrew:
   ```sh
   brew install python
   ```

## Setting Up the Service

### Step 1: Create and Configure the Python Script

1. **Create Directory and Script**:
   Create a directory for the script if it doesn't exist and move the script there.
   ```sh
   mkdir -p ~/url_converter
   ```

2. **Create the Script**:
   Create and edit `url_converter.py` file in the `~/url_converter` directory with the following content:

   ```python
   import http.server
   import urllib.parse
   import webbrowser

   class RequestHandler(http.server.BaseHTTPRequestHandler):
       def do_GET(self):
           parsed_path = urllib.parse.urlparse(self.path)
           query = urllib.parse.parse_qs(parsed_path.query)
           word = query.get('text', [''])[0]

           # Check if the query word is present
           if not word:
               self.send_response(400)
               self.send_header('Content-type', 'text/html')
               self.end_headers()
               self.wfile.write(b'Missing query text.')
               return

           # Encode the word
           encoded_word = urllib.parse.quote(word)

           # Build easydict URL
           easydict_url = f"easydict://query?text={encoded_word}"

           # Open easydict URL
           webbrowser.open(easydict_url)

           # Send response
           self.send_response(200)
           self.send_header('Content-type', 'text/html')
           self.end_headers()
           self.wfile.write(b'URL has been converted and opened.')

   def run(server_class=http.server.HTTPServer, handler_class=RequestHandler):
       server_address = ('', 8080)
       httpd = server_class(server_address, handler_class)
       print('Starting http server...')
       httpd.serve_forever()

   if __name__ == "__main__":
       run()
   ```

### Step 2: Configure Launchd to Run the Script at Startup

1. **Create Launchd Configuration File**:
   Create a directory for launch agents if it doesn't exist and create the configuration file.
   ```sh
   mkdir -p ~/Library/LaunchAgents
   nano ~/Library/LaunchAgents/com.user.urlconverter.plist
   ```

2. **Add the Following Content**:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
       <key>Label</key>
       <string>com.user.urlconverter</string>
       <key>ProgramArguments</key>
       <array>
           <string>/usr/local/bin/python3</string>
           <string>/Users/yourusername/url_converter/url_converter.py</string>
       </array>
       <key>RunAtLoad</key>
       <true/>
       <key>KeepAlive</key>
       <true/>
       <key>StandardOutPath</key>
       <string>/tmp/urlconverter.log</string>
       <key>StandardErrorPath</key>
       <string>/tmp/urlconverter.err</string>
   </dict>
   </plist>
   ```

   Replace `yourusername` with your actual username.

3. **Load and Start the Service**:
   ```sh
   launchctl load ~/Library/LaunchAgents/com.user.urlconverter.plist
   ```

4. **Verify the Service**:
   ```sh
   launchctl list | grep com.user.urlconverter
   ```

## Configuring Calibre to Use the Local Service

1. **Open Calibre**:
   - Launch the `Calibre` application.

2. **Access Preferences**:
   - Click on the "Preferences" button in the toolbar, or press `Ctrl+P` to open the Preferences window.

3. **Configure Lookup Plugin**:
   - In the Preferences window, find and click on "Lookup" under the "Advanced" section.
   - Under the "Lookup sources" tab, click on "Add a custom source".

4. **Add Custom Lookup Source**:
   - In the new window, fill in the following information:
     - **Name**: Enter a descriptive name, such as "Local Dictionary Service".
     - **Lookup URL**: Enter `http://localhost:8080/?text={word}`.

5. **Save Settings**:
   - Click "OK" to save the custom lookup source, then close the Preferences window.

## Usage

1. **Run the Python Script**:
   - Ensure the Python script is running by executing the following command in the terminal:
     ```sh
     python3 ~/url_converter/url_converter.py
     ```

2. **Use Calibre Lookup**:
   - In `Calibre`, when you select a word and use the lookup feature, the configured custom source will send a request to `http://localhost:8080/?text={word}`.
   - The local service will convert this request to `easydict://query?text={word}` and open it in the default web browser.

## Troubleshooting

- **Logs**: Check the logs at `/tmp/urlconverter.log` and `/tmp/urlconverter.err` for any issues.
- **Service Status**: Ensure the service is running by verifying its status with `launchctl list | grep com.user.urlconverter`.
- **Script Path**: Verify that the script path in the `plist` file is correct.
- **Python Path**: Ensure Python is installed and accessible at `/usr/local/bin/python3`.

## Contributing

Feel free to submit issues or pull requests for improvements and bug fixes.

## License

This project is licensed under the MIT License.