<p align="center"><a href="https://keepandroidopen.org/"><b>ANDROID WILL BECOME A LOCKED-DOWN PLATFORM</b></a></p>

# OPTIQON Voice

A tiny (2MB) Android dictation app. Point it at any OpenAI-compatible backend and type by speaking.

OPTIQON Voice does the same job as [WisprFlow](https://wisprflow.ai), except it costs nothing and is more private, connecting to your own models.

> IMPORTANT: This app was coded with AI. I am not an android developer, and I wouldn't be able to do this if it wasn't for AI. I try to enforce good practices, but any contributions or suggestions are very welcome. The app works pretty fine, is small, and gives me what I want.

> OPTIQON Voice is a rebrand of a fork of [pluja/sasayaki](https://github.com/pluja/sasayaki),
> licensed under GPLv3. See [LICENSE](LICENSE) and
> [docs/OPTIQON_VOICE_IDENTITY.md](docs/OPTIQON_VOICE_IDENTITY.md) for the full provenance and
> identity notes.

## Features

- A floating dot appears when the keyboard opens. Tap it to dictate, and it stops when you go quiet.
  - While you speak it widens into a timer with pause, cancel and profile buttons.
  - Tap the bubble while it is processing to drop the dictation.
- **Profiles** bundle a language, an ASR model, an LLM model and a writing style. Switch the active one from the bubble.
- **Post-processing**: a small model turns what you said into a clean written message.
  - "Uh... Yeah, let's meet at 8.. No sorry, at 9!" → "Let's meet at 9."
  - Per profile, set the punctuation and casing, how far the model may rewrite you, how much it should condense, and whether emoji are allowed.
  - OPTIQON Voice passes the app you are dictating into as context, so mail comes out in a different tone from a chat.
- Replacement rules fix the words your ASR keeps mangling. Plain text or regex, and each profile chooses which ones apply.
- Write your own post-processing prompts and switch them on per profile, alongside the built-in ones.
- History keeps past dictations, up to a limit you choose, and can be switched off. Word and time totals keep counting either way.
- The bubble comes back on its own after a reboot.

<table>
  <tr>
    <td><img src="/assets/screenshot01.png" width="250"></td>
    <td><img src="/assets/screenshot02.png" width="250"></td>
  </tr>
  <tr>
    <td align="center"><i>Main dashboard</i></td>
    <td align="center"><i>Floating bubble recording in-chat</i></td>
  </tr>
</table>

## Install

> This repository is being rebranded ahead of a move to `LarsAhls/optiqon-voice`. Until that move
> happens, releases and Obtainium updates are cut from this repository
> (`LarsAhls/sasayaki-Lars`); the links below point at the future `optiqon-voice` repository and
> will start resolving once that migration ships (a future, separate mission — see
> [docs/OPTIQON_VOICE_IDENTITY.md](docs/OPTIQON_VOICE_IDENTITY.md)).

Grab the latest APK from the [releases](https://github.com/LarsAhls/optiqon-voice/releases) page and install it. [Obtainium](https://github.com/ImranR98/Obtainium) can keep it updated for you:

<a href="https://apps.obtainium.imranr.dev/redirect?r=obtainium://app/{%22id%22:%22se.optiqon.voice%22,%22url%22:%22https://github.com/LarsAhls/optiqon-voice%22,%22author%22:%22LarsAhls%22,%22name%22:%22OPTIQON%20Voice%22,%22additionalSettings%22:%22{\%22about\%22:\%22A%20tiny%20dictation%20app.%20Connect%20any%20OpenAI-compatible%20backend%20and%20type%20by%20speaking.\%22}%22}"><img src="/assets/badge_obtainium.png" alt="Get it on Obtainium" height="60"></a>

You need an Android device and an ASR backend that speaks the OpenAI API. An LLM for post-processing is optional and worth it.

## Backends

Nothing here needs hosting: the app is the whole product, and it will talk to any OpenAI-compatible API. Run your own if you care about privacy and owning your data. If you would rather not, [ppq.ai](https://ppq.ai/) and [nano-gpt.com](https://nano-gpt.com) are reasonable choices when you trust them. If you want a fully private backend, check out [tinfoil](https://tinfoil.sh).

### Speech to text

Plenty of options exist, from [whisper.cpp](https://github.com/ggml-org/whisper.cpp) to [faster-whisper](https://github.com/SYSTRAN/faster-whisper). I run [speaches](https://github.com/speaches-ai/speaches).

For the model itself, [`deepdml/faster-whisper-large-v3-turbo-ct2`](https://huggingface.co/deepdml/faster-whisper-large-v3-turbo-ct2) beat everything else I tried across several European languages. On a GPU it is quick, and it holds up when you talk fast or even whisper.

Other options:

- The [whisper](https://huggingface.co/collections/openai/whisper-release) family, from tiny up to medium. For English only, the `en` models are smaller and sharper.
- [`Parakeet`](https://parakeettdt.com/) gives good results.
- [`moonshine`](https://github.com/moonshine-ai/moonshine) is small and multilingual, though I know of no OpenAI-compatible API for it.
- [`Voxtral Mini`](https://huggingface.co/mistralai/Voxtral-Mini-4B-Realtime-2602)

Setting your main languages in the app lowers the word error rate.

### LLM post-processing

Optional. A 2B or 4B model on a consumer GPU adds little delay and cleans the text up well.

I fine-tuned the 2B version of Qwen3.5 ([unsloth/Qwen3.5-2B](https://huggingface.co/unsloth/Qwen3.5-2B)) using the recipe in the [`fine-tuning`](https://github.com/LarsAhls/sasayaki-Lars/tree/main/fine-tuning#fine-tuning) directory. Generate a synthetic dataset in your language, then train with [`unsloth`](https://unsloth.ai/) on about 5GB of VRAM.

To serve it, I use [`llama-swap`](https://github.com/mostlygeek/llama-swap) over a llama.cpp backend. Plain [`llama.cpp`](https://github.com/ggml-org/llama.cpp), [`koboldcpp`](https://github.com/LostRuins/koboldcpp) and [LlamaFiles](https://github.com/mozilla-ai/llamafile) all work too.

The same providers are useful if you don't want to host your own ([ppq.ai](https://ppq.ai/) and [nano-gpt.com](https://nano-gpt.com)). If you want private inference, check out [tinfoil](https://tinfoil.sh).

Pick a model that follows instructions. Some will answer a dictated question instead of transcribing it, so that "what is the capital of France" becomes "Paris" in your chat box. The eval suite below catches that.

## Development

Build the release APK in Docker, which avoids installing an Android SDK or a matching JDK:

```sh
make build      # writes optiqon-voice-release.apk
```

Run the prompt tests, which need no network and no credentials:

```sh
./gradlew testDebugUnitTest
```

Two further suites measure the post-processing prompts against real models. They skip themselves unless `OPENAI_ENDPOINT` and `OPENAI_API_KEY` are set, so they never run by accident:

- `PostProcessingBenchmark` sends dictations in English, Spanish, French, Catalan and Italian, then checks whether self-corrections were applied, whether the text stayed in its original language, and how long each model took.
- `StyleControlDifferentialTest` verifies that moving a style control changes the output, which is how I found two settings that did nothing at all.

Both write a report under `app/build/reports/benchmark/`.

## License

GPLv3 — see [LICENSE](LICENSE).

## Provenance

OPTIQON Voice is a rebrand of a fork of [pluja/sasayaki](https://github.com/pluja/sasayaki),
released by its original author under GPLv3. This repository (`LarsAhls/sasayaki-Lars`) started
as that fork; the app is being renamed and re-identified as OPTIQON Voice while remaining GPLv3
open source. See [docs/OPTIQON_VOICE_IDENTITY.md](docs/OPTIQON_VOICE_IDENTITY.md) for the identity
principles that govern this rename, including what may still reference the upstream name.

### Historical note: the original name

Upstream's original name, ささやき (sasayaki), means "whisper" in Japanese — a nod picked by the
original author while learning the language. It no longer names this app, but the history is worth
keeping on record.
