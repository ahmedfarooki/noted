# Extracting and Transcribing Speech from a Video with Whisper

A recorded talk is easier to review when you can read and search what was said. This guide explains how to extract audio from a video and convert its speech into text using FFmpeg and OpenAI’s open-source Whisper. We will use `google-talk.mp4` as the example recording and run everything locally on a Mac.

## What voice extraction means

There are two separate steps. **Audio extraction** saves the video’s audio track as an audio file. **Transcription** converts the spoken words into written text.

Extracting the audio keeps the sounds in that track, including music and background noise. It does not isolate a particular person’s voice or clean up the recording. Whisper handles speech recognition; isolating speakers or removing noise requires a separate workflow.

## Tool licensing

Whisper is released under the MIT license. The supplied workflow also references FFmpeg, which is copyleft: its default license is LGPL v2.1 or later, and some builds use GPL v2 or later. Redistributing FFmpeg can require license notices, corresponding source, and, under LGPL, provisions for relinking modified versions; GPL builds impose GPL terms on covered derivative works. Check the exact build and its [official licensing information](https://ffmpeg.org/legal.html) before redistribution. This repository contains documentation only and does not bundle these tools.

## Install the tools

You need Python 3 and Homebrew installed on your Mac. Open Terminal and run:

```bash
brew install ffmpeg

python3 -m venv whisper-env
source whisper-env/bin/activate
python -m pip install --upgrade pip
python -m pip install --upgrade openai-whisper
```

FFmpeg reads and converts media files. Whisper performs the transcription. The virtual environment keeps Whisper’s Python dependencies separate from other projects. These installation steps follow the [official Whisper setup instructions](https://github.com/openai/whisper#setup).

For the commands below, open Terminal in the folder containing `google-talk.mp4`, with the virtual environment activated.

## Extract the audio track

To create a standalone WAV file, run:

```bash
ffmpeg -i "google-talk.mp4" \
  -map 0:a:0 -vn \
  -ac 1 -ar 16000 -c:a pcm_s16le \
  "google-talk.wav"
```

This selects the first audio track, excludes video, and creates a mono WAV recording at a 16 kHz sample rate. It leaves the original MP4 intact. If the video contains multiple audio tracks, confirm that the first track contains the conversation you want. The options are documented in the [FFmpeg reference](https://ffmpeg.org/ffmpeg.html).

Listen to the WAV briefly before transcribing to confirm that it contains the expected audio.

## Convert speech into text

For an English-language talk, run:

```bash
whisper "google-talk.wav" \
  --model base.en \
  --language English \
  --task transcribe \
  --device cpu \
  --fp16 False \
  --output_format all \
  --output_dir transcripts
```

The CPU settings provide a straightforward starting point without requiring a dedicated GPU. The first run downloads the selected model. After the software and model are installed, this local workflow processes the recording on your computer without uploading it to a transcription service.

The `base.en` model is a useful starting choice for English. To try a larger model, replace it with `small.en`. Larger models trade additional processing time and resources for potential accuracy improvements; results depend on the recording and hardware. See [Whisper’s model comparison](https://github.com/openai/whisper#available-models-and-languages).

## Skip the separate audio file when convenient

If you only need a transcript, Whisper’s FFmpeg-based audio loader can read the MP4 directly. Use the same command above, replacing `google-talk.wav` with `google-talk.mp4`. You do not need to create a WAV first. See the [audio-loading implementation](https://github.com/openai/whisper/blob/main/whisper/audio.py).

## Use the generated files

The `transcripts` folder will contain several formats, including:

| File | Useful for |
|---|---|
| `google-talk.txt` | Reading, searching, and copying the transcript |
| `google-talk.srt` | Subtitles with timestamps |
| `google-talk.vtt` | Web captions with timestamps |
| `google-talk.json` | Processing transcription data in code |

Whisper’s [command-line implementation](https://github.com/openai/whisper/blob/main/whisper/transcribe.py) documents the output formats and processing options.

Review the transcript against the recording, especially names, technical terms, numbers, and overlapping speech. Standard Whisper transcription does not automatically label speakers as “interviewer” and “candidate.”

Once reviewed, the text becomes a searchable record of the talk. Keep the SRT or VTT alongside it when you want to return to the original recording at a particular moment.
