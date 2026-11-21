# EAS / SAME Decoder for SDR#

V1 preview 1.0.1.0 · SDR# 1922 Beta · Windows x64 · .NET 10 / Airspy VDK.

Automatically watches the tuned channel for EAS/SAME headers during live reception or IQ playback. The compact panel shows the latest encoder ID and decode confidence. Details and reception history provide the full transmitted header and the evidence behind the rating. No station database or automatic transmitter identification is included.

## Install

1. Finish any recordings and close the target SDR# instance.
2. Extract the **preview ZIP** into the SDR# installation folder containing `SDRSharp.exe`, preserving the ZIP's `Plugins` folder. The resulting DLL is `Plugins\SDRSharp.SAME\SDRSharp.SAME.dll`.
3. Keep one copy of that plugin DLL under the Plugins tree. Move old copies to a backup outside Plugins before updating.
4. Open SDR#, then **Menu → Plugins → Decoders : EAS / SAME Decoder**.
5. Leave **Enabled** checked, start the receiver or IQ playback, and tune the desired station.

The preview ZIP is the package to share with testers. The source ZIP is for developers. Neither package contains SDR#, the VDK's reference libraries, recordings, receiver connections, or personal settings.

## Listen and decode

The plugin listens automatically while Enabled, even when its panel is hidden behind another docked tab. **DSB, binaural audio, and the AM Co-Channel Canceller can stay enabled.** It copies audio for analysis without changing your listening setup. Muting the speakers does not disable decoding.

For AM/DSB with binaural enabled, it automatically uses SDR#'s stereo PreAF audio hook, after the IF/CCC processing, AGC and audio filtering, before speaker volume and mute. It decodes left and right separately rather than mixing them together, so opposite-polarity audio cannot cancel out in a mono sum. The status reads **Listening · binaural L/R**. For other supported configurations it uses the mono detector output, before later audio processing. Audio filtering and other processing before the selected hook can still affect the decode.

It monitors **one tuned channel**, not every station within the received IQ band. AM or DSB can be used for MW broadcasts, WFM for broadcast FM, and NFM for NOAA Weather Radio. The IF/channel filter still affects decoding: an AM/DSB filter must pass both 1562.5 and 2083.3 Hz audio tones; approximately 6–10 kHz total bandwidth is a useful starting point. Use an appropriate NFM/WFM filter for those services. Deep fades, co-channel interference, squelch, and narrow filters can remove information the decoder cannot recover.

RAW mode and bypassed demodulation do not provide the demodulated audio needed for decoding. Listening needs at least a 6 kS/s input rate. Very high rates above 5 MS/s are outside this preview's supported range.

## Compact panel

- **Enabled** controls automatic listening.
- **Status** distinguishes listening, stopped radio, unsupported audio configuration, and decoder failures.
- **Encoder ID** shows the latest transmitted identifier. A damaged message's readable ID is explicitly marked **partial**.
- **Confidence** summarizes agreement between separately received header bursts.
- **Details…** opens the event/originator codes, issue time, validity interval, coverage codes, all captured headers, and confidence evidence. **Copy Details** prepares a report for sharing.
- **History…** opens the latest 200 receptions. Select an entry and choose Details, or double-click it.
- **Auto layout / Horizontal / Vertical** controls the arrangement. Auto uses the vertical layout in a narrow dock. Drag and stack the tool window using SDR#'s normal docking controls; a strip below the spectrum or a side dock both work.

The latest result stays visible until another reception replaces it. Repeated headers update one reception entry. End-of-message markers update that entry rather than replacing its station ID. If only an ending is recovered, the result says **End marker only**. No sound alerts, toolbar badges, or automatic tab switching are used.

History is retained in SDR#'s per-window settings when SDR# closes normally. The history limit is 200 receptions; old entries are removed as new entries arrive. Selecting a read-only text field pauses that field's refresh while it has focus so its text can be copied.

## Confidence and partial results

| Rating | Evidence |
|---|---|
| Very high | Three identical, complete header bursts. |
| High | Two identical, complete header bursts. |
| Unconfirmed | One complete header with no matching complete repeat yet. |
| Conflicting | Complete headers disagree; Details preserves each variant. If two agree, the rating says High with a conflicting-repeat note. |
| Partial | No complete header accepted. Raw damaged text is available; readable ID fragments are clues. |
| End marker only | `NNNN` recovered without a usable opening header. |

These are evidence-based labels, not calibrated probabilities. SAME has no CRC; a well-formed single header can still contain incorrect characters. This V1 does not guess missing characters or reconstruct a complete header by voting across damaged copies. It tries multiple timing/byte alignments and preserves the best recovered text from each physical burst. An unused eighth ASCII bit is ignored as the protocol permits; that is not a guessed character repair.

Hearing the same physical burst in both binaural channels counts as **one** header, so stereo cannot inflate the confidence rating. Details identifies the channel(s) used. If the channels produce different complete headers for the same burst, both variants are retained, the disagreement is flagged, and that burst is excluded from the matching-repeat count.

Encoder IDs can belong to a message's originator or retransmitting encoder. Coverage codes describe the alert area, not necessarily the transmitter location. Use those clues with your own station list. No online lookup, network access, telemetry, or maintained station list is required.

## Times and location codes

The reception/decoded timestamp is the **PC's processing time in UTC**, including during recorded-IQ playback. It is an approximate host/audio timestamp, not a calibrated RF timestamp. It does not currently use the recording's original date/time. The header's **issue time** is shown separately as transmitted: day of year and UTC hour/minute. SAME does not transmit a year, so V1 does not invent one.

State, territory, marine-area, and subdivision names are translated where mapped. County/area numbers are retained numerically; V1 does not bundle a county-name database. All original location codes remain available. Unknown event or geography codes remain visible as unmapped codes.

## Preview testing and limitations

The supplied WAV recordings and converted channel-A copies of dual-channel Linrad IQ are used for validation inside SDR# 1922. Baseline AM results: WJR and WWL each produced three identical complete headers from both audio clips and real IQ playback. The original WFIL audio test produced two matching complete headers (High confidence). WONE remained Partial with readable ID fragments; 540 kHz and 1320 kHz did not yield complete headers. All five Linrad recordings were checked through channel-A IQ playback. Controlled NFM and WFM versions of the supplied WWL audio each yielded three matching headers. Protocol checks covered the unused eighth bit, 31 location fields, invalid fields, conflicting repeats, arbitrary block boundaries, unchanged input samples, and a 30-second noise negative control. Release build: 0 warnings, 0 errors. Converted IQ files preserve one receiver channel; this plugin does not implement Linrad's dual-channel diversity processing or open Linrad RAW files itself. Use SDR#'s Baseband File Player with a compatible IQ WAV.

Version 1.0.1 adds automatic AM/DSB binaural support. Native DSB/binaural/CCC playback recovered all three matching WWL headers (Very high), and two matching complete WONE headers plus a damaged repeat (High). WJR recovered one complete header and one damaged repeat (Unconfirmed); with CCC disabled, a left/right disagreement was detected and the result correctly remained Conflicting. These results demonstrate compatibility, not equal recovery for every listening configuration. Separate stereo checks covered opposite-polarity left/right audio, a silent left or right channel, one/two/three physical headers, unchanged input samples, and conflicting channel decodes. Mode, binaural and CCC settings remained under the user's control.

The decoder is a first pass at nominal-rate SAME AFSK. It does not yet track large transmitter/audio clock errors or perform soft combining of damaged repeats. Weak-signal results will vary with the station, receiver, interference, and IF processing. Controlled NFM/WFM modulation checks exercise those detector paths; actual FM/NWR field recordings and longer unattended use still need broader testing.

Audio and decoded-result queues are bounded. If the decoder cannot keep up, it stops and reports an overflow instead of silently joining audio across a gap. Disable and re-enable listening to restart. Retuning, mode/filter changes, and stream-rate changes reset synchronization. During IQ playback, allow the header to replay from its beginning after seeking; arbitrary seeks/loops have not been comprehensively validated in this version.

## Helpful tester reports

Include the SDR# and plugin versions, receiver/source, detector mode, binaural and CCC settings, IF bandwidth, live versus recorded playback, and **Copy Details** output. A short original audio or IQ recording containing the opening header bursts is particularly useful. Report missed decodes, unexpected history entries, and any case where damaged text appears complete.

## Build from source

Place the source `SAME` folder alongside `lib` in an Airspy VDK checkout. Install the .NET 10 SDK and prepare SDR# as described by the VDK, then run:

```powershell
dotnet build .\SAME\SDRSharp.SAME.csproj -c Release -p:SDRSharpDir="C:\Path\To\SDRSharp\"
```

Close the deployment target first. SDR# supplies the runtime libraries; SDK references are compile-time only. No external NuGet package is needed. The source ZIP excludes validation instrumentation and SDR# runtime copies.

Architecture: the selected audio hook copies samples and their tuning/rate context into preallocated native packets. A bounded SDR# circular buffer feeds a worker. SDR#'s native `RealResampler` supplies a 12.5 kS/s analysis stream per channel; matched tone correlations feed parallel bit/byte alignments. A stereo burst merger combines observations of the same physical burst. Burst results cross an explicitly bounded single-producer/single-consumer queue. The UI groups repetitions and maintains history. No file I/O, decoder work, audio modification, or steady allocation occurs in the audio callbacks.

Protocol reference: [47 CFR §11.31, EAS protocol](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-11/subpart-B/section-11.31), also available through [Cornell LII](https://www.law.cornell.edu/cfr/text/47/11.31).

## License

See `LICENSE.txt` for the Airspy VDK license and its Airspy ecosystem restriction. This package is an SDR# plugin; include the license when redistributing it.


