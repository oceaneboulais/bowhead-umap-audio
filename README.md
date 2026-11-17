# bowhead-umap-audio

Audio clips for the interactive UMAP page
(<https://www.oceaneboulais.net/interactive_whale_calls_3d.html>, "Click a point" tab).
One clip per spectrogram that the page can show (11,770 of the 100,000 training spectrograms).

## What the clips are

- **Real recordings**, not reconstructions: 3.0 s cut from the continuous 1 kHz DASAR (Directional Autonomous
  Seafloor Acoustic Recorder) recordings made on the Alaskan North Slope (Sites 3 and 5, 2008-2014).
- Mono, **1000 Hz sample rate**, so they contain frequencies up to 500 Hz. Bowhead calls (25-500 Hz) are at the
  low end of what laptop speakers reproduce; use headphones or the page's speed-up control.
- Each clip is peak-normalised (to 0.9 of full scale) and mean-removed, so **loudness is not comparable between clips**.

## Files

| file | content |
|---|---|
| `tile_NNN.bin` | 64 clips, little-endian `int16`, 3000 samples each (384 000 bytes). Clip `slot` is tile `slot // 64`, index `slot % 64` (same numbering as the page's spectrogram tiles). Unused trailing slots are zero. |
| `audio.json` | `sr`, `n_samples`, `per_tile`, `n_slots`, `n_tiles`, and `flags_b64` (one byte per slot: 0 no audio, 1 weak spectrogram match, 2 good spectrogram match). |

## How a clip was aligned to its spectrogram

An event's filename timestamp is only a rough guide to where its spectrogram sits in the day-long recording (about
1 s off on average). For each event a 6 s window was read around the filename position, and the window was slid until
the energy-versus-time profile of its own spectrogram (256-point FFT, 26 ms steps, the same as the stored image)
best matched the stored spectrogram's profile (Pearson r). The 3 s clip is cut at the best position.

- Median match r = 0.92; r >= 0.6 for 96.5 % of clips (flag 2). The remaining 3.5 % (flag 1) are kept but their
  position is approximate.
- Check: for the 5,422 events whose position was independently confirmed by peak frequency, the chosen position agrees
  with the independently-confirmed one to within a median of 0.012 s (inter-quartile range about +/-0.04 s).
- The spectrogram on the page spans the first 2.7 s of each 3.0 s clip.

Code: `bowhead/benchmark/build_umap_audio_tiles.py` and `bowhead/eval/raw_audio_loader.py` in
<https://github.com/oceaneboulais/BowheadWhaleDetector>.

## Provenance

Recordings collected under Shell Exploration and Production Company's Beaufort Sea monitoring program
(Greeneridge Sciences). Event catalogue and labels: Boulais et al., in preparation (JASA). A licence file has not
been added; add one that matches the data-sharing terms before wider redistribution.
