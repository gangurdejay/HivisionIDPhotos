# API Documentation

## Before You Start: Launch the Backend Service

Before making API requests, please run the backend service:

```bash
python deploy_api.py
```

<br>

## Interface Function Descriptions

### 1. Generate ID Photo (Transparent Background)

Interface Name: `idphoto`

The `Generate ID Photo` interface logic involves sending an RGB image and receiving a standard ID photo and a high-definition ID photo:

- **High-Definition ID Photo**: An ID photo made according to the aspect ratio of `size`, with the filename being `output_image_dir` appended with `_hd` suffix.
- **Standard ID Photo**: A photo with dimensions equal to `size`, scaled from the high-definition ID photo, with the filename being `output_image_dir`.

It should be noted that both generated photos are transparent (RGBA four-channel images). To produce a complete ID photo, the following `Add Background Color` interface is also required.

> Q: Why is this design used?  
> A: In actual products, users often need to frequently switch background colors to preview effects. Providing a transparent background image and allowing the front-end JavaScript code to synthesize the color offers a better user experience.

### 2. Add Background Color

Interface Name: `add_background`

The `Add Background Color` interface logic involves sending an RGBA image, adding a background color based on `color`, and synthesizing a JPG image.

### 3. Generate 6-inch Layout Photo

Interface Name: `generate_layout_photos`

The `Generate 6-inch Layout Photo` interface logic involves sending an RGB image (usually an ID photo after adding a background color), arranging the photos according to `size`, and then generating a 6-inch layout photo.

<br>

## 1️⃣ Python Requests Method

### 1. Generate ID Photo (Transparent Background)

```python
import requests

url = "http://127.0.0.1:8080/idphoto"
input_image_path = "images/test.jpg"

files = {"input_image": open(input_image_path, "rb")}
data = {"height": 413, "width": 295}

response = requests.post(url, files=files, data=data).json()

# response is a JSON dictionary containing status, image_base64_standard, and image_base64_hd
print(response)
```

### 2. Add Background Color

```python
import requests

url = "http://127.0.0.1:8080/add_background"
input_image_path = "test.png"

files = {"input_image": open(input_image_path, "rb")}
data = {"color": '638cce', 'kb': None}

response = requests.post(url, files=files, data=data).json()

# response is a JSON dictionary containing status and image_base64
print(response)
```

### 3. Generate 6-inch Layout Photo

```python
import requests

url = "http://127.0.0.1:8080/generate_layout_photos"
input_image_path = "test.jpg"

files = {"input_image": open(input_image_path, "rb")}
data = {"height": 413, "width": 295, "kb": 200}

response = requests.post(url, files=files, data=data).json()

# response is a JSON dictionary containing status and image_base64
print(response)
```

<br>

## 2️⃣ Python Script Request Method

```bash
python requests_api.py -u <URL> -t <TYPE> -i <INPUT_IMAGE_DIR> -o <OUTPUT_IMAGE_DIR> [--height <HEIGHT>] [--width <WIDTH>] [-c <COLOR>] [-k <KB>]
```

### Parameter Descriptions

#### Basic Parameters

- `-u`, `--url`

  - **Description**: The URL of the API service.
  - **Default Value**: `http://127.0.0.1:8080`

- `-t`, `--type`

  - **Description**: The type of API request, with optional values being `idphoto`, `add_background`, and `generate_layout_photos`. They represent ID photo creation, transparent image background addition, and layout photo generation, respectively.
  - **Default Value**: `idphoto`

- `-i`, `--input_image_dir`

  - **Description**: The path of the input image.
  - **Required**: Yes
  - **Example**: `./input_images/photo.jpg`

- `-o`, `--output_image_dir`
  - **Description**: The path to save the image.
  - **Required**: Yes
  - **Example**: `./output_images/processed_photo.jpg`

#### Optional Parameters

- `--height`

  - **Description**: The height of the output size for the standard ID photo.
  - **Default Value**: 413

- `--width`

  - **Description**: The width of the output size for the standard ID photo.
  - **Default Value**: 295

- `-c`, `--color`

  - **Description**: Adds a background color to the transparent image, in Hex format (e.g., #638cce), only effective when the type is `add_background`.
  - **Default Value**: `638cce`

- `-k`, `--kb`
  - **Description**: The KB value of the output photo, only effective when the type is `add_background` or `generate_layout_photos`, and no setting is made when the value is None.
  - **Default Value**: `None`
  - **Example**: `50`

### 1. Generate ID Photo (Transparent Background)

```bash
python requests_api.py  \
    -u http://127.0.0.1:8080 \
    -t idphoto \
    -i ./photo.jpg \
    -o ./idphoto.png \
    --height 413 \
    --width 295
```

### 2. Add Background Color

```bash
python requests_api.py  \
    -u http://127.0.0.1:8080  \
    -t add_background  \
    -i ./idphoto.png  \
    -o ./idphoto_with_background.jpg  \
    -c 638cce  \
    -k 50
```

### 3. Generate 6-inch Layout Photo

```bash
python requests_api.py  \
    -u http://127.0.0.1:8080  \
    -t generate_layout_photos  \
    -i ./idphoto_with_background.jpg  \
    -o ./layout_photo.jpg  \
    --height 413  \
    --width 295 \
    -k 200
```

### Request Failure Scenarios

- The request fails if more than one face is detected in the photo.
