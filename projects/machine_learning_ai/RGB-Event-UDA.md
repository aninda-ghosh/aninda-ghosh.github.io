## Bridging the Gap between Events and Frames through Unsupervised Domain Adaptation

### Project description
This project was initially created by the UZG Perception Group for leveraging the labelled dataset in RGB space to be used for the event domain datasets.

We modified the codebase to use the `ResNext50` based bottleneck blocks to make the training faster and improved the accuracy further for the classification task on N-Caltech101 Dataset.

[Presentation](/pdf/PIR%20Presentation.pdf) | [Project Page](https://github.com/aninda-ghosh/RGB-Event-UDA)

### Model Creation Code

Attributing to lesser depth to trainable parameters in ResNext models, it resulted into   

```python
attribute_channels = 8

model = torch.hub.load('pytorch/vision:v0.10.0', 'resnext50_32x4d', pretrained=True)

# Shared Encoder Layers
self.front_end_shared = list(model.children())[4][2]

# Front End Sensor A
self.front_end_sensor_a = StyleEncoder(
  self.settings.input_channels_a, 
  self.front_end_shared,
  attribute_channels, 
  self.settings.use_decoder_a
  )

# Front End Sensor B
self.front_end_sensor_b = StyleEncoder(
  self.settings.input_channels_b, 
  self.front_end_shared,
  attribute_channels, 
  self.settings.use_decoder_b
  )

# Discriminator
self.discriminator = ContentDiscriminator(nr_channels=256)

self.models_dict = {
  "front_sensor_a": self.front_end_sensor_a,
  "front_sensor_b": self.front_end_sensor_b,
  "front_shared": self.front_end_shared,
  "dis": self.discriminator
  }

# Task Backend
if self.settings.use_task_a or self.settings.use_task_b:
    self.task_backend = nn.Sequential(*(list(model.children())[5:-1] + [nn.Flatten(), nn.Linear(2048, 101)]))
    self.models_dict["back_end"] = self.task_backend

# Decoders
if self.settings.use_decoder_a:
    self.decoder_sensor_a = StyleDecoder(
      input_c=256, 
      output_c=self.settings.input_channels_a,
      attribute_channels=attribute_channels,
      sensor_name=self.settings.sensor_a_name
      )
    self.models_dict["decoder_sensor_a"] = self.decoder_sensor_a

if self.settings.use_decoder_b:
    self.decoder_sensor_b = StyleDecoder(
      input_c=256, 
      output_c=2,
      attribute_channels=attribute_channels,
      sensor_name=self.settings.sensor_b_name
      )
    self.models_dict["decoder_sensor_b"] = self.decoder_sensor_b

# Cross Refinement Networks
if self.settings.cross_refinement_a:
    self.cross_refinement_net_a = refinement_net.StyleRefinementNetwork(
      input_c=2,
      output_c=self.settings.input_channels_a,
      sensor=self.settings.sensor_a_name,
      channel_list=[16, 8],
      last_layer_pad=1,
      device=self.device)
    self.refinement_discr_a = CrossDiscriminator(
      input_dim=self.settings.input_channels_a, 
      n_layer=6
      )
    self.models_dict["cross_refinement_net_sensor_a"] = self.cross_refinement_net_a
    self.models_dict["refinement_discr_sensor_a"] = self.refinement_discr_a

if self.settings.cross_refinement_b:
    self.cross_refinement_net_b = refinement_net.StyleRefinementNetwork(
      input_c=2,
      output_c=self.settings.input_channels_b,
      sensor=self.settings.sensor_b_name,
      channel_list=[16, 8],
      last_layer_pad=1,
      device=self.device
      )
    self.refinement_discr_b = CrossDiscriminator(
      input_dim=self.settings.input_channels_b, 
      n_layer=6
      )

    self.models_dict["cross_refinement_net_sensor_b"] = self.cross_refinement_net_b
    self.models_dict["refinement_discr_sensor_b"] = self.refinement_discr_b
```
