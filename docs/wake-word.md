# Wake Word

The typical workflow for interacting with a voice assistant is to first activate it with a "wake" or "hot" word, then provide your voice command. Rhasspy supports listening for a wake word with one of several systems.

You can also wake Rhasspy up using the [HTTP API](usage.md#http-api) by POST-ing to `/api/listen-for-command`. Rhasspy will immediately wake up and [start listening](command-listener.md) for a voice command.

The following table summarizes the key characteristics of each wake word system:

| System                                    | Performance | Training to Customize | Online Sign Up          |
| ------                                    | ----------- | -----------------     | ----------------------- |
| [porcupine](wake-word.md#porcupine)       | excellent   | yes, offline          | no                      |
| [snowboy](wake-word.md#snowboy)           | good        | yes, online           | yes                     |
| [pocketsphinx](wake-word.md#pocketsphinx) | poor        | no                    | no                      |
| [precise](wake-word.md#mycroft-precise)   | moderate    | yes, offline          | no                      |

## Porcupine

Listens for a wake word with [porcupine](https://github.com/Picovoice/Porcupine). This system has the best performance out of the box. If you want a custom wake word, however, you will need to re-run their optimizer tool every 30 days.

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "porcupine",
  "porcupine": {
    "library_path": "porcupine/libpv_porcupine.so",
    "model_path": "porcupine/porcupine_params.pv",
    "keyword_path": "porcupine/porcupine.ppn",
    "sensitivity": 0.5
  }
},

"rhasspy": {
  "listen_on_start": true
}
```

There are a lot of [keyword files](https://github.com/Picovoice/Porcupine/tree/master/resources/keyword_files) available for download. Use the `linux` platform if you're on desktop/laptop (`amd64`) and the `raspberrypi` platform if you're using a Raspberry Pi (`armhf`/`aarch64`). The `.ppn` files should go in the `porcupine` directory inside your profile (referenced by `keyword_path`).

If you want to create a custom wake word, you will need to use the [Picovoice Console](https://github.com/Picovoice/porcupine#picovoice-console). **NOTE**: the generated keyword file is only valid for 30 days, though you can always just re-run the optimizer.

See `rhasspy.wake.PorcupineWakeListener` for details.

## Snowboy

Listens for one or more wake words with [snowboy](https://snowboy.kitt.ai). This system has the good performance out of the box, but requires an online service to train.

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "snowboy",
  "hermes": {
    "wakeword_id": "default"
  },
  "snowboy": {
    "model": "snowboy/snowboy.umdl",
    "audio_gain": 1,
    "sensitivity": "0.5",
    "apply_frontend": false
  }
},

"rhasspy": {
  "listen_on_start": true
}
```

If your hotword model has multiple embedded hotwords (such as `jarvis.umdl`), the "sensitivity" parameter should contain sensitivities for each embedded hotword separated by commas (e.g., "0.5,0.5").

Visit [the snowboy website](https://snowboy.kitt.ai) to train your own wake word model (requires linking to a GitHub/Google/Facebook account). This *personal* model with end with `.pmdl`, and should go in your profile directory. Then, set `wake.snowboy.model` to the name of that file.

You also have the option of using a pre-train *universal* model (`.umdl`) from [Kitt.AI](https://github.com/Kitt-AI/snowboy/tree/master/resources/models).

### Multiple Wake Words

You can have `snowboy` listen for multiple wake words with different models, each with their own settings. You will need to download each model file to the `snowboy` directory in your profile.

For example, to use both the `snowboy.umdl` and `jarvis.umdl` models, add this to your profile:

```json
"wake": {
  "system": "snowboy",
  "snowboy": {
    "model": "snowboy/snowboy.umdl,snowboy/jarvis.umdl",
    "model_settings": {
      "snowboy/snowboy.umdl": {
        "sensitivity": "0.5",
        "audio_gain": 1,
        "apply_frontend": false
      },
      "snowboy/jarvis.umdl": {
        "sensitivity": "0.5,0.5",
        "audio_gain": 1,
        "apply_frontend": false
      }
    }
  }
}
```

Make sure to include all models you want in the `model` setting (separated by commas). Each model may have different settings in `model_settings`. If a setting is not present, the default values under `snowboy` will be used.

See `rhasspy.wake.SnowboyWakeListener` for details.

## Pocketsphinx

Listens for a [keyphrase](https://cmusphinx.github.io/wiki/tutoriallm/#using-keyword-lists-with-pocketsphinx) using [pocketsphinx](https://github.com/cmusphinx/pocketsphinx). This is the most flexible wake system, but has the worst performance in terms of false positives/negatives.

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "pocketsphinx",
  "pocketsphinx": {
    "keyphrase": "okay rhasspy",
    "threshold": 1e-30,
    "chunk_size": 960
  }
},

"rhasspy": {
  "listen_on_start": true
}
```

Set `wake.pocketsphinx.keyphrase` to whatever you like, though 3-4 syllables is recommended. Make sure to [train](training.md) and restart Rhasspy whenever you change the keyphrase.

The `wake.pocketsphinx.threshold` should be in the range 1e-50 to 1e-5. The smaller the number, the less like the keyphrase is to be observed. At least one person has written a script to [automatically tune the threshold](https://medium.com/@PankajB96/automatic-tuning-of-keyword-spotting-thresholds-a27256869d31).

See `rhasspy.wake.PocketsphinxWakeListener` for details.

## Mycroft Precise

Listens for a wake word with [Mycroft Precise](https://github.com/MycroftAI/mycroft-precise). It requires training up front, but can be done completely offline!

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "precise",
  "precise": {
    "model": "model-name-in-profile.pb",
    "sensitivity": 0.5,
    "trigger_level": 3,
    "chunk_size": 2048
  }
},

"rhasspy": {
  "listen_on_start": true
}
```

Follow [the instructions from Mycroft AI](https://github.com/MycroftAI/mycroft-precise/wiki/Training-your-own-wake-word#how-to-train-your-own-wake-word) to train your own wake word model. When you're finished, place **both** the `.pb` and `.pb.params` files in your profile directory, and set `wake.precise.model` to the name of the `.pb` file.

See `rhasspy.wake.PreciseWakeListener` for details.

## MQTT/Hermes

Subscribes to the `hermes/hotword/<WAKEWORD_ID>/detected` topic, and wakes Rhasspy up when a message is received ([Hermes protocol](https://docs.snips.ai/reference/hermes)). This allows Rhasspy to use the wake word functionality in [Snips.AI](https://snips.ai/).

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "hermes",
  "hermes": {
    "wakeword_id": "default"
  }
},


"rhasspy": {
  "listen_on_start": true
},

"mqtt": {
  "enabled": true,
  "host": "localhost",
  "username": "",
  "port": 1883,
  "password": "",
  "site_id": "default"
}
```

Adjust the `mqtt` configuration to connect to your MQTT broker.
Set `mqtt.site_id` to match your Snips.AI siteId and `wake.hermes.wakeword_id` to match your Snips.AI wakewordId.

See `rhasspy.wake.HermesWakeListener` for details.

## Command

Calls a custom external program to listen for a wake word, only waking up Rhasspy when the program exits.

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "command",
  "command": {
    "program": "/path/to/program",
    "arguments": []
  }
},

"rhasspy": {
  "listen_on_start": true
}
```

When Rhasspy starts, your program will be called with the given arguments. Once your program detects the wake word, it should print it to standard out and exit. Rhasspy will call your program again when it goes back to sleep. If the empty string is printed, Rhasspy will **not** wake up and your program will be called again.

The following environment variables are available to your program:

* `$RHASSPY_BASE_DIR` - path to the directory where Rhasspy is running from
* `$RHASSPY_PROFILE` - name of the current profile (e.g., "en")
* `$RHASSPY_PROFILE_DIR` - directory of the current profile (where `profile.json` is)

See [sleep.sh](https://github.com/synesthesiam/rhasspy/blob/master/bin/mock-commands/sleep.sh) for an example program.

See `rhasspy.wake.CommandWakeListener` for details.

## Dummy

Disables wake word functionality.

Add to your [profile](profiles.md):

```json
"wake": {
  "system": "dummy"
}
```

See `rhasspy.wake.DummyWakeListener` for details.
