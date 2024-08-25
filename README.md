# CodeProject.AI Home Assistant Object Detection custom component

This component is a direct port of the [HASS-plate-recognizer](https://github.com/robmarkcole/HASS-plate-recognizer) component by [Robin Cole](https://github.com/robmarkcole). This component provides AI-based Object Detection capabilities using [CodeProject.AI Server](https://codeproject.com/ai). 

 [CodeProject.AI Server](https://codeproject.com/ai) is a service which runs either in a Docker container or as a Windows Service and exposes various an API for many AI inferencing operations via a REST API. The Object Detection capabilities use the [YOLO](https://arxiv.org/pdf/1506.02640.pdf) algorithm as implemented by Ultralytics and others. It can identify 80 different kinds of objects by default, but custom models are also available that focus on specific objects such as animals, license plates or objects typically encountered by home webcams. CodeProject.AI Server is free, locally installed, and can run without an external internet connection, is is comatible with Windows, Linux, macOS. It can run on Raspberry Pi, and supports CUDA and embedded Intel GPUs.

On the machine in which you are running CodeProject.AI server, either ensure the service is running, or if using Docker, [start a Docker container](https://www.codeproject.com/ai/docs/why/running_in_docker.html#launching-a-container). 

This integration adds an image processing entity where the state of the entity is the number of license plates found in a processed image. Information about the vehicle which has the license plate is provided in the entity attributes, and includes the license plate number, and confidence (in a scale 0 to 1) in this prediction. For each vehicle an `platerecognizer.vehicle_detected` event is fired, containing the same information just listed. 

**Note** this integration does NOT automatically process images, it is necessary to call the `image_processing.scan` service to trigger processing.

## Home Assistant setup
Place the `custom_components` folder in your configuration directory (or add its contents to an existing `custom_components` folder). Then configure as below:

```yaml
image_processing:
  - platform: platerecognizer
    watched_plates:
      - kbw46ba
      - kfab726
    save_file_folder: /config/images/platerecognizer/
    save_timestamped_file: True
    always_save_latest_file: True
    server: http://yoururl:8080/v1/vision/alpr/

    source:
      - entity_id: camera.yours
```
Then, **restart** your Home Assistant

Configuration variables:
- **watched_plates**: (Optional) A list of number plates to watch for, which will identify a plate even if a couple of digits are incorrect in the prediction (fuzzy matching). If configured this adds an attribute to the entity with a boolean for each watched plate to indicate if it is detected.
- **save_file_folder**: (Optional) The folder to save processed images to. Note that folder path should be added to [whitelist_external_dirs](https://www.home-assistant.io/docs/configuration/basic/)
- **save_timestamped_file**: (Optional, default `False`, requires `save_file_folder` to be configured) Save the processed image with the time of detection in the filename.
- **always_save_latest_file**: (Optional, default `False`, requires `save_file_folder` to be configured) Always save the last processed image, no matter there were detections or not.
- **server**: (Optional, requires a [paid plan](https://platerecognizer.com/pricing/) Provide a local server address to use [On-Premise SDK](https://docs.platerecognizer.com/#on-premise-sdk)
- **source**: Must be a camera.

<p align="center">
<img src="https://github.com/robmarkcole/HASS-plate-recognizer/blob/main/docs/card.png" width="400">
</p>

<p align="center">
<img src="https://github.com/robmarkcole/HASS-plate-recognizer/blob/main/docs/main.png" width="800">
</p>

<p align="center">
<img src="https://github.com/robmarkcole/HASS-plate-recognizer/blob/main/docs/event.png" width="800">
</p>

## Making a sensor for individual plates
If you have configured `watched_plates` you can create a binary sensor for each watched plate, using a [template sensor](https://www.home-assistant.io/integrations/template/) as below, which is an example for plate `kbw46ba`:

```yaml
sensor:
  - platform: template
    sensors:
      plate_recognizer:
        friendly_name: "kbw46ba"
        value_template: "{{ state_attr('image_processing.platerecognizer_1', 'watched_plates').kbw46ba }}"
```

Depending on your license plate, you may recieve an template error due to variables not being able to start with a number. if so, here is another method to create the template sensor:
```yaml
sensor:
  - platform: template
    sensors:
      plate_recognizer:
        friendly_name: "kbw46ba"
        value_template: "{{ state_attr("image_processing.platerecognizer_1", "watched_plates")["kbw46ba"] }}"
```


## Video of usage
Checkout this excellent video of usage from [Everything Smart Home](https://www.youtube.com/channel/UCrVLgIniVg6jW38uVqDRIiQ)

[![](http://img.youtube.com/vi/t-XxCrdj_94/0.jpg)](http://www.youtube.com/watch?v=t-XxCrdj_94 "")
