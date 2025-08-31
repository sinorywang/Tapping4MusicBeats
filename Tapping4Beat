from flask import Flask, render_template_string, request, jsonify, send_file
import time
import io
import os
from werkzeug.utils import secure_filename

app = Flask(__name__)
ALLOWED_EXTENSIONS = {'mp3', 'wav', 'ogg'}
UPLOAD_FOLDER = 'uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)

timestamps = []
final_bpm = "N/A"
start_time = None
last_tap_time = None
current_audio_file = None


def allowed_file(filename):
    return '.' in filename and \
        filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS


@app.route('/upload', methods=['POST'])
def upload_file():
    global current_audio_file
    if 'file' not in request.files:
        return jsonify({'error': 'No file part'}), 400
    file = request.files['file']
    if file.filename == '':
        return jsonify({'error': 'No selected file'}), 400
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(filepath)

        current_audio_file = filename

        return jsonify({
            'success': True,
            'filename': filename,
            'url': f'/uploads/{filename}',
            'status': f'Song "{filename}" uploaded successfully! Press Space or the "Tap" button to start.'
        })
    return jsonify({'error': 'Invalid file type'}), 400


@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_file(os.path.join(app.config['UPLOAD_FOLDER'], filename))


@app.route('/')
def index():
    html_content = """
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Sinory's BPM Recorder</title>
        <style>
            body {
                font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                background-color: #f4f4f4;
                display: flex;
                justify-content: center;
                align-items: center;
                min-height: 100vh;
                margin: 0;
            }
            .container {
                background-color: #fff;
                padding: 40px;
                border-radius: 12px;
                box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
                text-align: center;
            }
            h1 {
                color: #333;
                margin-bottom: 20px;
            }
            #status {
                font-size: 1.2em;
                color: #555;
                margin-bottom: 10px;
            }
            #bpm {
                font-size: 2.5em;
                color: #007bff;
                font-weight: bold;
                margin-bottom: 25px;
            }
            .controls button {
                padding: 15px 30px;
                font-size: 1.1em;
                border: none;
                border-radius: 8px;
                cursor: pointer;
                margin: 10px;
                transition: background-color 0.3s ease;
            }
            #tap-button {
                background-color: #28a745;
                color: white;
            }
            #tap-button:hover {
                background-color: #218838;
            }
            #clear-button {
                background-color: #dc3545;
                color: white;
            }
            #clear-button:hover {
                background-color: #c82333;
            }
            #save-button {
                background-color: #007bff;
                color: white;
            }
            #save-button:hover {
                background-color: #0056b3;
            }
            #upload-section {
                margin-bottom: 20px;
                padding: 10px;
                border: 1px solid #ccc;
                border-radius: 8px;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>Sinory's BPM Recorder</h1>

            <div id="upload-section">
                <input type="file" id="audio-file" accept="audio/*">
                <button id="upload-button">Upload Song</button>
            </div>

            <audio id="player" controls style="display:none;"></audio>

            <div id="status">That's How it Starts: Plz Click the"Tap"Botton or Spacebar</div>
            <div id="bpm">Current BPM: N/A</div>
            <div class="controls">
                <button id="tap-button" disabled>Tap</button>
                <button id="clear-button">Clear All</button>
                <button id="save-button">Store the data</button>
            </div>
        </div>

        <script>
            const tapButton = document.getElementById('tap-button');
            const clearButton = document.getElementById('clear-button');
            const saveButton = document.getElementById('save-button');
            const uploadButton = document.getElementById('upload-button');
            const audioFile = document.getElementById('audio-file');
            const statusDiv = document.getElementById('status');
            const bpmDiv = document.getElementById('bpm');
            const player = document.getElementById('player');

            let isFirstTap = true;
            let tapEnabled = false;
            let isCountingDown = false;
            let countdownInterval = null;
            let lastTapTime = 0;
            const debounceThreshold = 150;

            async function recordTap() {
                const response = await fetch('/tap', { method: 'POST' });
                const data = await response.json();
                statusDiv.textContent = data.status;
                bpmDiv.textContent = data.bpm;
            }

            function startCountdown() {
                if (!tapEnabled || isCountingDown) return;

                isCountingDown = true;
                tapButton.disabled = true; 

                let countdown = 5;
                statusDiv.textContent = `Countdown: ${countdown} seconds`;

                countdownInterval = setInterval(() => {
                    countdown--;
                    statusDiv.textContent = `Countdown: ${countdown} seconds`;

                    if (countdown <= 0) {
                        clearInterval(countdownInterval);
                        isCountingDown = false;

                        player.play();
                        recordTap();

                        tapButton.disabled = false;
                        statusDiv.textContent = 'Song has started. Keep tapping!';
                    }
                }, 1000);
            }

            async function uploadFile() {
                const file = audioFile.files[0];
                if (!file) {
                    statusDiv.textContent = 'Please select an audio file!';
                    return;
                }

                statusDiv.textContent = 'Uploading...';

                const formData = new FormData();
                formData.append('file', file);

                try {
                    const response = await fetch('/upload', {
                        method: 'POST',
                        body: formData
                    });
                    const data = await response.json();

                    if (data.success) {
                        statusDiv.textContent = data.status;
                        player.src = data.url;
                        tapButton.disabled = false;
                        tapEnabled = true;
                        isFirstTap = true;

                        await fetch('/clear', { method: 'POST' });

                    } else {
                        statusDiv.textContent = 'Upload failed: ' + data.error;
                    }
                } catch (error) {
                    statusDiv.textContent = 'Connection error, unable to upload file.';
                }
            }

            async function clearTaps() {
                player.pause();
                player.currentTime = 0;

                if (isCountingDown) {
                    clearInterval(countdownInterval);
                    isCountingDown = false;
                    tapButton.disabled = true;
                }

                isFirstTap = true;
                tapEnabled = false;

                const response = await fetch('/clear', { method: 'POST' });
                const data = await response.json();
                statusDiv.textContent = 'Please upload a song (MP3/WAV)!';
                bpmDiv.textContent = data.bpm;
            }

            async function saveData() {
                player.pause();

                const response = await fetch('/save', { method: 'GET' });
                const blob = await response.blob();
                const url = window.URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.style.display = 'none';
                a.href = url;
                a.download = 'timestamps_and_final_bpm.txt';
                document.body.appendChild(a);
                a.click();
                window.URL.revokeObjectURL(url);

                statusDiv.textContent = 'Data saved.';
            }

            uploadButton.addEventListener('click', uploadFile);
            tapButton.addEventListener('click', () => {
                if (isFirstTap) {
                    startCountdown();
                    isFirstTap = false;
                } else {
                    recordTap();
                }
            });
            clearButton.addEventListener('click', clearTaps);
            saveButton.addEventListener('click', saveData);

            document.addEventListener('keydown', (e) => {
                const currentTime = Date.now();
                if (e.code === 'Space' && tapEnabled && !isCountingDown && (currentTime - lastTapTime > debounceThreshold)) {
                    e.preventDefault();
                    lastTapTime = currentTime;
                    if (isFirstTap) {
                        startCountdown();
                        isFirstTap = false;
                    } else {
                        recordTap();
                    }
                }
            });

            tapButton.disabled = true;
            statusDiv.textContent = 'Please upload a song (MP3/WAV)!';
        </script>
    </body>
    </html>
    """
    return render_template_string(html_content)


@app.route('/tap', methods=['POST'])
def tap_endpoint():
    global timestamps, final_bpm, start_time, last_tap_time

    current_absolute_time = time.time()

    if start_time is None:
        start_time = current_absolute_time
        relative_time = 0.0
        status_text = "Tapping Started. Please continue!"
        timestamps.append(relative_time)

    else:
        relative_time = current_absolute_time - start_time
        if (current_absolute_time - last_tap_time) > 3:
            timestamps = []
            final_bpm = "N/A"
            relative_time = current_absolute_time - start_time
            status_text = "You've rested for too long. Restarted calculation."
            timestamps.append(relative_time)
        else:
            timestamps.append(relative_time)
            intervals = [timestamps[i + 1] - timestamps[i] for i in range(len(timestamps) - 1)]

            if len(intervals) > 0:
                avg_interval = sum(intervals) / len(intervals)
                bpm = 60 / avg_interval
                final_bpm = f"{bpm:.1f}"

            status_text = f"Total Tapping: {len(timestamps)}"

    last_tap_time = current_absolute_time

    return jsonify(status=status_text, bpm=f"Current BPM: {final_bpm}")


@app.route('/clear', methods=['POST'])
def clear_endpoint():
    global timestamps, final_bpm, start_time, last_tap_time
    timestamps = []
    final_bpm = "N/A"
    start_time = None
    last_tap_time = None
    return jsonify(status="All Recordings are deleted.", bpm="Current BPM: N/A")


@app.route('/save', methods=['GET'])
def save_endpoint():
    if not timestamps:
        return jsonify(error="No Data To Store!"), 400

    header = "Time (s)\n"
    data_lines = [f"{t:.6f}" for t in timestamps]

    final_line = f"\nFinal BPM: {final_bpm}"

    str_data = header + "\n".join(data_lines) + final_line

    binary_data = str_data.encode('utf-8')
    buffer = io.BytesIO(binary_data)
    buffer.seek(0)

    return send_file(
        buffer,
        mimetype='text/plain',
        as_attachment=True,
        download_name='timestamps_and_final_bpm.txt'
    )


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
