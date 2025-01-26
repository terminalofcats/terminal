<h1><div align="center">
Cat Terminal
</div></h1>

Cat Terminal is an open source Python framework for building voice and multimodal conversational agents. It handles the complex orchestration of AI services, network transport, audio processing, and multimodal interactions, letting you focus on creating engaging experiences.

## What you can build

- **Voice Assistants**: [Natural, real-time conversations with AI](https://demo.dailybots.ai/)
- **Interactive Agents**: Personal coaches and meeting assistants
- **Multimodal Apps**: Combine voice, video, images, and text
- **Creative Tools**: [Story-telling experiences](https://storytelling-chatbot.fly.dev/) and social companions
- **Business Solutions**: [Customer intake flows](https://www.youtube.com/watch?v=lDevgsp9vn0) and support bots

## See it in action

<p float="left">
    <a href="https://github.com/pipecat-ai/pipecat/tree/main/examples/simple-chatbot"><img src="https://raw.githubusercontent.com/pipecat-ai/pipecat/main/examples/simple-chatbot/image.png" width="280" /></a>&nbsp;
    <a href="https://github.com/pipecat-ai/pipecat/tree/main/examples/storytelling-chatbot"><img src="https://raw.githubusercontent.com/pipecat-ai/pipecat/main/examples/storytelling-chatbot/image.png" width="280" /></a>
    <br/>
    <a href="https://github.com/pipecat-ai/pipecat/tree/main/examples/translation-chatbot"><img src="https://raw.githubusercontent.com/pipecat-ai/pipecat/main/examples/translation-chatbot/image.png" width="280" /></a>&nbsp;
    <a href="https://github.com/pipecat-ai/pipecat/tree/main/examples/moondream-chatbot"><img src="https://raw.githubusercontent.com/pipecat-ai/pipecat/main/examples/moondream-chatbot/image.png" width="280" /></a>
</p>

## Key features

- **Voice-first Design**: Built-in speech recognition, TTS, and conversation handling
- **Flexible Integration**: Works with popular AI services (OpenAI, ElevenLabs, etc.)
- **Pipeline Architecture**: Build complex apps from simple, reusable components
- **Real-time Processing**: Frame-based pipeline architecture for fluid interactions
- **Production Ready**: Enterprise-grade WebRTC and Websocket support

## Getting started

You can get started with Cat Terminal running on your local machine, then move your agent processes to the cloud when you’re ready. You can also add a 📞 telephone number, 🖼️ image output, 📺 video input, use different LLMs, and more.

```shell
# Install the module
pip install Cat-Terminal-ai

# Set up your environment
cp dot-env.template .env
```

To keep things lightweight, only the core framework is included by default. If you need support for third-party AI services, you can add the necessary dependencies with:

```shell
pip install "Cat-Terminal-ai[option,...]"
```

## A simple voice agent running locally

Here is a very basic Cat Terminal bot that greets a user when they join a real-time session. We'll use [Daily](https://daily.co) for real-time media transport, and [Cartesia](https://cartesia.ai/) for text-to-speech.

```python
import asyncio

from catterminal.frames.frames import EndFrame, TextFrame
from catterminal.pipeline.pipeline import Pipeline
from catterminal.pipeline.task import PipelineTask
from catterminal.pipeline.runner import PipelineRunner
from catterminal.services.cartesia import CartesiaTTSService
from catterminal.transports.services.daily import DailyParams, DailyTransport

async def main():
  # Use Daily as a real-time media transport (WebRTC)
  transport = DailyTransport(
    room_url=...,
    token="", # leave empty. Note: token is _not_ your api key
    bot_name="Bot Name",
    params=DailyParams(audio_out_enabled=True))

  # Use Cartesia for Text-to-Speech
  tts = CartesiaTTSService(
    api_key=...,
    voice_id=...
  )

  # Simple pipeline that will process text to speech and output the result
  pipeline = Pipeline([tts, transport.output()])

  # Create Cat Terminal processor that can run one or more pipelines tasks
  runner = PipelineRunner()

  # Assign the task callable to run the pipeline
  task = PipelineTask(pipeline)

  # Register an event handler to play audio when a
  # participant joins the transport WebRTC session
  @transport.event_handler("on_first_participant_joined")
  async def on_first_participant_joined(transport, participant):
    participant_name = participant.get("info", {}).get("userName", "")
    # Queue a TextFrame that will get spoken by the TTS service (Cartesia)
    await task.queue_frame(TextFrame(f"Hello there, {participant_name}!"))

  # Register an event handler to exit the application when the user leaves.
  @transport.event_handler("on_participant_left")
  async def on_participant_left(transport, participant, reason):
    await task.queue_frame(EndFrame())

  # Run the pipeline task
  await runner.run(task)

if __name__ == "__main__":
  asyncio.run(main())
```

Run it with:

```shell
python app.py
```

Daily provides a prebuilt WebRTC user interface. While the app is running, you can visit at `https://<yourdomain>.daily.co/<room_url>` and listen to the bot say hello!

## WebRTC for production use

WebSockets are fine for server-to-server communication or for initial development. But for production use, you’ll need client-server audio to use a protocol designed for real-time media transport. (For an explanation of the difference between WebSockets and WebRTC, see [this post.](https://www.daily.co/blog/how-to-talk-to-an-llm-with-your-voice/#webrtc))

One way to get up and running quickly with WebRTC is to sign up for a Daily developer account. Daily gives you SDKs and global infrastructure for audio (and video) routing. Every account gets 10,000 audio/video/transcription minutes free each month.

Sign up [here](https://dashboard.daily.co/u/signup) and [create a room](https://docs.daily.co/reference/rest-api/rooms) in the developer Dashboard.

## Hacking on the framework itself

_Note that you may need to set up a virtual environment before following the instructions below. For instance, you might need to run the following from the root of the repo:_

```shell
python3 -m venv venv
source venv/bin/activate
```

From the root of this repo, run the following:

```shell
pip install -r dev-requirements.txt
```

This will install the necessary development dependencies. Also, make sure you install the git pre-commit hooks:

```shell
pre-commit install
```

The hooks will just save you time when you submit a PR by making sure your code follows the project rules.

To use the package locally (e.g. to run sample files), run:

```shell
pip install --editable ".[option,...]"
```

The `--editable` option makes sure you don't have to run `pip install` again and you can just edit the project files locally.

If you want to use this package from another directory, you can run:

```shell
pip install "path_to_this_repo[option,...]"
```

### Running tests

From the root directory, run:

```shell
pytest
```

## Setting up your editor

This project uses strict [PEP 8](https://peps.python.org/pep-0008/) formatting via [Ruff](https://github.com/astral-sh/ruff).

### Emacs

You can use [use-package](https://github.com/jwiegley/use-package) to install [emacs-lazy-ruff](https://github.com/christophermadsen/emacs-lazy-ruff) package and configure `ruff` arguments:

```elisp
(use-package lazy-ruff
  :ensure t
  :hook ((python-mode . lazy-ruff-mode))
  :config
  (setq lazy-ruff-format-command "ruff format")
  (setq lazy-ruff-check-command "ruff check --select I"))
```

`ruff` was installed in the `venv` environment described before, so you should be able to use [pyvenv-auto](https://github.com/ryotaro612/pyvenv-auto) to automatically load that environment inside Emacs.

```elisp
(use-package pyvenv-auto
  :ensure t
  :defer t
  :hook ((python-mode . pyvenv-auto-run)))
```

### Visual Studio Code

Install the
[Ruff](https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff) extension. Then edit the user settings (_Ctrl-Shift-P_ `Open User Settings (JSON)`) and set it as the default Python formatter, and enable formatting on save:

```json
"[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true
}
```

### PyCharm

`ruff` was installed in the `venv` environment described before, now to enable autoformatting on save, go to `File` -> `Settings` -> `Tools` -> `File Watchers` and add a new watcher with the following settings:

1. **Name**: `Ruff formatter`
2. **File type**: `Python`
3. **Working directory**: `$ContentRoot$`
4. **Arguments**: `format $FilePath$`
5. **Program**: `$PyInterpreterDirectory$/ruff`

## Getting help

➡️ [Reach us on X](https://x.com/Cat_Terminal_)
