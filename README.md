# Sinory's BPM Recorder

A simple web-based application for tapping the beats per minute (BPM) of a song. Users can upload an audio file and tap along to the music using the spacebar or a button. The app calculates and displays the current BPM based on the taps.

## Features

- **Audio Upload**: Supports MP3, WAV, and OGG file formats.
- **BPM Calculation**: Calculates and displays the BPM in real-time as you tap.
- **Tap Control**: Tap to the beat using the "Tap" button or the **spacebar**.
- **Long Press Debounce**: Ignores repeated key presses from a long-held spacebar to ensure accurate tapping.
- **Data Management**: Clear all recorded taps or save the tapping data and final BPM to a `.txt` file.
- **User-Friendly Interface**: A clean and simple interface for a seamless tapping experience.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

You need to have Python and `pip` installed on your system.

### Installation

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/YourUsername/Sinory-BPM-Recorder.git](https://github.com/YourUsername/Sinory-BPM-Recorder.git)
    cd Sinory-BPM-Recorder
    ```

2.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv venv
    # On Windows
    venv\\Scripts\\activate
    # On macOS/Linux
    source venv/bin/activate
    ```

3.  **Install the required libraries:**
    ```bash
    pip install Flask
    ```

## Usage

1.  **Run the application:**
    ```bash
    python Tapping4Beat.py
    ```
    The application will start on `http://127.0.0.1:5000` (or `http://0.0.0.0:5000` if you're running it on a server).

2.  **Open the application in your browser:**
    Navigate to the URL provided in your terminal.

3.  **Upload a song:**
    Click the "Upload Song" button and select an audio file (MP3, WAV, or OGG).

4.  **Start tapping:**
    Wait for the 5-second countdown to finish, and then start tapping along with the beat using the **spacebar** or the "Tap" button.

5.  **View and save results:**
    The BPM will be displayed in real-time. You can click "Store the data" to save the timestamps and final BPM to a text file.

## File Structure

- `Tapping4Beat.py`: The main Flask application file containing all the backend logic and the HTML template.
- `uploads/`: A directory where uploaded audio files are temporarily stored.

## Contributing

Feel free to submit issues or pull requests to improve the project. Any suggestions for new features or enhancements are welcome!

## License

This project is licensed under the MIT License - see the `LICENSE.md` file for details.
