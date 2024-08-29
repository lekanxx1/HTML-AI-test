const express = require('express');
const path = require('path');
const { SpeechClient } = require('@google-cloud/speech');
const cors = require('cors');

const app = express();
const port = process.env.PORT || 3000;

// Serve static files from the 'public' directory
app.use(express.static(path.join(__dirname, 'public')));

// Enable CORS
app.use(cors());

// Speech-to-Text client
const speechClient = new SpeechClient();

app.post('/transcribe', express.json(), async (req, res) => {
  try {
    const [response] = await speechClient.recognize({
      config: {
        encoding: 'LINEAR16',
        sampleRateHertz: 16000,
        languageCode: 'en-US',
      },
      audio: {
        content: req.body.audioData,
      },
    });

    const transcription = response.results
      .map(result => result.alternatives[0].transcript)
      .join('\n');

    res.json({ transcription });
  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ error: 'Transcription failed' });
  }
});

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
