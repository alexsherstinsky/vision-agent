## VisionAgent
VisionAgent is a library that helps you utilize agent frameworks to generate code to
solve your vision task. Check out our discord for updates and roadmaps! The fastest
way to test out VisionAgent is to use our web application which you can find [here](https://va.landing.ai/).

## Installation
```bash
pip install vision-agent
```

```bash
export ANTHROPIC_API_KEY="your-api-key"
export GEMINI_API_KEY="your-api-key"
```

> **_NOTE:_** We found using both Anthropic Claude-3.7 and Gemini-2.0-Flash-Exp to be provide the best performance for VisionAgent. If you want to use a different LLM provider or only one, see 'Using Other LLM Providers' below.

## Documentation

[VisionAgent Library Docs](https://landing-ai.github.io/vision-agent/)

## Examples
### Counting cans in an image
You can run VisionAgent in a local Jupyter Notebook [Counting cans in an image](https://github.com/landing-ai/vision-agent/blob/main/examples/notebooks/counting_cans.ipynb)

### Generating code
You can use VisionAgent to generate code to count the number of people in an image:
```python
from vision_agent.agent import VisionAgentCoderV2
from vision_agent.models import AgentMessage

agent = VisionAgentCoderV2(verbose=True)
code_context = agent.generate_code(
    [
        AgentMessage(
            role="user",
            content="Count the number of people in this image",
            media=["people.png"]
        )
    ]
)

with open("generated_code.py", "w") as f:
    f.write(code_context.code + "\n" + code_context.test)
```

### Using the tools directly
VisionAgent produces code that utilizes our tools. You can also use the tools directly.
For example if you wanted to detect people in an image and visualize the results:
```python
import vision_agent.tools as T
import matplotlib.pyplot as plt

image = T.load_image("people.png")
dets = T.countgd_object_detection("person", image)
# visualize the countgd bounding boxes on the image
viz = T.overlay_bounding_boxes(image, dets)

# save the visualization to a file
T.save_image(viz, "people_detected.png")

# display the visualization
plt.imshow(viz)
plt.show()
```

You can also use the tools for running on video files:
```python
import vision_agent.tools as T

frames_and_ts = T.extract_frames_and_timestamps("people.mp4")
# extract the frames from the frames_and_ts list
frames = [f["frame"] for f in frames_and_ts]

# run the countgd tracking on the frames
tracks = T.countgd_sam2_video_tracking("person", frames)
# visualize the countgd tracking results on the frames and save the video
viz = T.overlay_segmentation_masks(frames, tracks)
T.save_video(viz, "people_detected.mp4")
```

## Using Other LLM Providers
You can use other LLM providers by changing `config.py` in the `vision_agent/configs`
directory. For example to change to Anthropic simply just run:
```bash
cp vision_agent/configs/anthropic_config.py vision_agent/configs/config.py
```

You can also modify the existing `config.py` file yourself to use a different LLM
provider, for example if you wanted to change the planner from Anthropic inside
`config.py` to OpenAI you would replace this code:
```python
    planner: Type[LMM] = Field(default=AnthropicLMM)
    planner_kwargs: dict = Field(
        default_factory=lambda: {
            "model_name": "claude-3-7-sonnet-20250219",
            "temperature": 0.0,
            "image_size": 768,
        }
    )
```

with this code:

```python
    planner: Type[LMM] = Field(default=OpenAILMM)
    planner_kwargs: dict = Field(
        default_factory=lambda: {
            "model_name": "gpt-4o-2024-11-20",
            "temperature": 0.0,
            "image_size": 768,
            "image_detail": "low",
        }
    )
```

> **_NOTE:_** VisionAgent moves fast and we are constantly updating and changing the library. If you have any questions or need help, please reach out to us on our discord channel.
