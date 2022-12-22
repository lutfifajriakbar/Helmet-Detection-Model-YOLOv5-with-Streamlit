# Sample application on Streamlit
This folder contains an example application on Streamlit to visualize the work of YOLOv5.

[Streamlit](https://docs.streamlit.io/en/stable/getting_started.html) is a framework that allows you to create an application for visualizing data or demonstrating the capabilities of a model. The framework is designed to quickly build a UI, but does not allow you to personalize page elements using CSS. To customize the appearance of the interface, you need to use another framework, for example, Dash Plotly

[YOLOv5](https://github.com/ultralytics/yolov5) - CNN for object recognition. The [version of the model] (https://pytorch.org/hub/ultralytics_yolov5/) from the Torch Hub library is used, trained to detect 80 object classes from the MS COCO dataset. The list of classes can be found in the `config.py` file

## Working example
Full demo can be found on [youtube](https://youtu.be/f_gbRXk6V0Y)
![caption](content/demo.gif)

## Requirements
Performance tested on Ubuntu 20.04 Python 3.8. Application is expected to run on Python 3.6-3.9

## Application files
* `app.py` - script to run the application
* `config.py` - script containing constants

## Run
Being in the `dashboards/streamlit_guide` folder, execute
```
pip install -r requirements.txt
streamlit run app.py
```
and follow the link http://localhost:8501

Or via docker
```
docker build -t st .
docker run -p 8501:8501st
```
**Note:** When using docker, Streamlit will display the External URL and Network URL in the terminal. When you navigate through them, the webcam will not work due to the HTTP protocol (requires HTTPS). But if you follow the link http://localhost:8501, then the problem will be solved

## Implementation details
### Streamlit
Streamlit allows you to display text, images, video, audio, graphics, dataframes on the application page. It is also possible to set up a stream from a webcam. You can learn more about all data types supported for output in [documentation](https://docs.streamlit.io/en/stable/api.html).

**Adding Components**

Interface elements are added to the page in the order in which the corresponding functions were called:
```
import streamlit as st
st.title('Streamlit app')
st.markdown('Streamlit is **_really_ cool**.')
if st.button('Say hello'):
     st.write('Why hello there')
else:
     st.write('Goodbye')
```
![example](content/streamlit_simple_app.png)

All elements added in this way will be placed in the main part of the page. However, by adding `sidebar` when creating the element, it will be added to the sidebar widget:
```
...
if st.sidebar.button('Say hello'):
     st.write('Why hello there')
else:
     st.write('Goodbye')
```
![example](content/simple_app_sidebar.png)

**Caching**

When the state of any control changes, the entire code of the script with the application will be executed again. To avoid re-execution of computationally expensive operations, Streamlit uses caching. To enable function caching, you must specify the [streamlit.cache()](https://docs.streamlit.io/en/stable/api.html#streamlit.cache) decorator:
```
@st.cache(max_entries=2)
def get_yolo5(model_type='s'):
     ...
```
To decide whether the data stored in the cache can be used, Streamlit checks:
1. Input parameters
2. Function body
3. Values of global variables used in the function body

If there is an object in the cache that matches the current state of the function on all three counts, then it will be used. By default, the cache capacity is unlimited, but you can set a limit with the `max_entries` argument.

When working with the cache, you must remember that for the input and output data in the cache, a link to them will be stored. Therefore, any change to this data in the code after calling the cached function results in a change in their values in the cache. Therefore, it is necessary to create copies of the input and output data:
```
# get_preds is a cacheable function
result = get_preds(img)
result_copy = result.copy() #for lists - deepcopy(list)
img_copy = img.copy()
# result_copy and img_copy can be used without fear of changing result and img
```
You can read more about caching in [documentation](https://docs.streamlit.io/en/stable/caching.html)

###streamlit-webrtc
To stream audio and video over the network, including to web cameras, you must use the [streamlit-webrtc] library (https://github.com/whitphx/streamlit-webrtc).

The implementation of streaming video from a webcam can be found in the `app.py` file. The `VideoTransformer` class allows you to specify the processing of each frame in the `transform()` function. The `webrtc_streamer()` function is used to initialize the stream. In order to pass new values of global variables to `VideoTransformer` when updating the application script, you must declare them as class attributes and when updating the script, pass:
```
WEBRTC_CLIENT_SETTINGS = ClientSettings(
         media_stream_constraints={"video": True, "audio": False},
     )

ctx = webrtc_streamer(
         key="examp"
