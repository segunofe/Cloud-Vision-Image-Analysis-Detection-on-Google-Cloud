# Cloud Vision Image Analysis Pipeline

A hands-on project that stores images in **Google Cloud Storage** and sends them to the **Google Cloud Vision API** over REST (JSON via `cURL`) to detect labels, web entities, faces, landmarks, and objects.

> **Note:** This project uses Google's pre-trained Vision API. It demonstrates working with cloud vision services and interpreting model outputs, not training a custom model. It follows the Google Cloud Skills Boost lab *Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API*.

## What it does

| Feature | Vision API type | What it returns |
|---|---|---|
| Label detection | `LABEL_DETECTION` | Descriptive labels for the image, each with a confidence score |
| Web detection | `WEB_DETECTION` | Related web entities, matching pages, and visually similar images |
| Face detection | `FACE_DETECTION` | A bounding box, facial landmarks, head angles, and emotion likelihoods for each face |
| Landmark detection | `LANDMARK_DETECTION` | Landmark name, coordinates, and where it appears in the image |
| Object localization | `OBJECT_LOCALIZATION` | Named objects with normalized bounding boxes |

## Workflow

```
Local image  ->  Cloud Storage bucket  ->  request.json  ->  Vision API (REST)  ->  JSON response
```

1. Upload an image to a Cloud Storage bucket.
2. Describe the request in `request.json` (image location and the feature to run).
3. Send it with `cURL` to the `images:annotate` endpoint.
4. Read the JSON response, including confidence scores and bounding boxes.

## Prerequisites

- A Google Cloud project with billing enabled
- The **Cloud Vision API** enabled
- An API key restricted to the Cloud Vision API
- Access to Cloud Shell (or a terminal with `curl`)

## Setup

**1. Create an API key** under *APIs & Services > Credentials*, restricted to the Cloud Vision API, then store it as an environment variable:

```bash
export API_KEY=<YOUR_API_KEY>
```

**2. Create a bucket and upload an image.** Use fine-grained access control, then upload your image (for example `donuts.png`).

**3. Make the image readable by the API.** For this demo, the lab grants `allUsers` the *Reader* role on the object. This makes the image public, so use only non-sensitive images (see [Security notes](#security-notes)).

## Usage

Create `request.json`, replacing `PROJECT_ID-bucket` with your bucket name:

```json
{
  "requests": [
    {
      "image": {
        "source": { "gcsImageUri": "gs://PROJECT_ID-bucket/donuts.png" }
      },
      "features": [
        { "type": "LABEL_DETECTION", "maxResults": 10 }
      ]
    }
  ]
}
```

Send the request:

```bash
curl -s -X POST -H "Content-Type: application/json" \
  --data-binary @request.json \
  "https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}" \
  -o label_detection.json && cat label_detection.json
```

To try another feature, change the `type` in `request.json`:

- **Web detection:** `WEB_DETECTION`
- **Faces:** `FACE_DETECTION` (upload an image with people in it)
- **Landmarks:** `LANDMARK_DETECTION` (upload a photo of a well-known landmark)
- **Objects:** `OBJECT_LOCALIZATION` (this can also take a public image URL using `imageUri` instead of `gcsImageUri`)

## Example response (label detection)

Shape of the output (values vary by image):

```json
{
  "responses": [
    {
      "labelAnnotations": [
        { "mid": "/m/01wydv", "description": "Beignet", "score": 0.95, "topicality": 0.95 },
        { "mid": "/m/02wbm",  "description": "Food",    "score": 0.94, "topicality": 0.94 }
      ]
    }
  ]
}
```

## Reading the output

- **`description` / `name`:** what the API thinks it found.
- **`score`:** confidence from 0 to 1. Higher means more certain.
- **`mid`:** an ID that maps to Google's Knowledge Graph.
- **`boundingPoly`:** the region of the image containing a face, landmark, or object. For object localization the coordinates are *normalized* (0 to 1) relative to image size.
- **Emotion likelihoods (faces):** ratings such as `joyLikelihood: LIKELY` or `angerLikelihood: VERY_UNLIKELY`. These are categories, not exact probabilities.

## Screenshots

_Add your own screenshots here, for example your bucket, `request.json`, and the JSON output for each feature._

```
docs/
  bucket.png
  label-detection-output.png
  face-detection-output.png
```

## Security notes

- **Never commit your API key.** Keep it in an environment variable, and add any `.env` file to `.gitignore`.
- Restrict the key to the Cloud Vision API only.
- Making a bucket object public exposes it to anyone with the link. Use non-sensitive sample images, and remove public access or delete the bucket when finished.
- Delete or rotate the API key after the project.

## What I learned

- How to structure and send REST requests to a cloud vision service
- How Cloud Storage links images to API calls
- How to interpret confidence scores, bounding boxes, and emotion likelihoods
- How results differ across image types

## Ideas for next steps

- Add **text detection (OCR)**, logo detection, and SafeSearch, which the Vision API also supports
- Wrap the `cURL` calls in a Python script that processes a folder of images in batch
- Store results in BigQuery for analysis
- Compare API output with a custom model (for example, a CNN trained on a defect dataset) for inspection use cases

## Acknowledgements

Based on the Google Cloud Skills Boost lab *Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API*. Sample images are provided by the lab and are not included in this repository.

## Technologies

Google Cloud Platform, Cloud Storage, Cloud Vision API, REST, JSON, cURL, Cloud Shell
