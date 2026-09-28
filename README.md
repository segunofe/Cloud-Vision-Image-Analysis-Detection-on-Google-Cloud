# Cloud Vision Image Analysis Pipeline

A hands-on project that stores images in **Google Cloud Storage** and sends them to the **Google Cloud Vision API** over REST (JSON via `cURL`) to detect labels, web entities, faces, landmarks, and objects.

Image for face detection 
<img width="960" height="960" alt="selfie1" src="https://github.com/user-attachments/assets/bf61537d-19ef-4b70-8b3e-b19224d8237c" />

Image for landmark detection 
<img width="2877" height="3596" alt="city" src="https://github.com/user-attachments/assets/17d7295f-6e48-49c1-9d78-e45e3082ecd3" />


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

<img width="1227" height="572" alt="Screenshot 2026-09-28 103025" src="https://github.com/user-attachments/assets/72a4af56-8047-433a-9e70-d043b6710dc4" />

<img width="1600" height="1200" alt="donuts" src="https://github.com/user-attachments/assets/68eb4a4f-1c8b-4f1d-b112-563a695d40d5" />
<img width="1720" height="748" alt="Screenshot 2026-09-28 140747" src="https://github.com/user-attachments/assets/df327592-bf19-47f1-800e-f6f4739d38bd" />



**3. Make the image readable by the API.** For this demo, I grant `allUsers` the *Reader* role on the object. This makes the image public, so use only non-sensitive images (see [Security notes](#security-notes)).


<img width="1276" height="832" alt="Screenshot 2026-09-28 103216" src="https://github.com/user-attachments/assets/d9bbf391-899b-4c3d-8e8f-cc2f067caed3" />


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




<img width="1842" height="741" alt="Screenshot 2026-09-28 100335" src="https://github.com/user-attachments/assets/7eef5a13-90d2-4605-b3e6-29629d5a8d26" />

  label-detection-output.png
<img width="1386" height="757" alt="Screenshot 2026-09-28 104420" src="https://github.com/user-attachments/assets/c95230e5-24d8-41f0-9bd3-1ec7d5818728" />


<img width="1096" height="687" alt="Screenshot 2026-09-28 103929" src="https://github.com/user-attachments/assets/c1e68f63-9485-41b8-8be1-8c028e3f39ad" />
<img width="828" height="542" alt="Screenshot 2026-09-28 104020" src="https://github.com/user-attachments/assets/1ca44777-ba57-4b90-b671-41a4108cebaa" />
<img width="776" height="528" alt="Screenshot 2026-09-28 104048" src="https://github.com/user-attachments/assets/a2c885a2-5f4b-4e55-be29-1a93501ee684" />
<img width="906" height="573" alt="Screenshot 2026-09-28 104109" src="https://github.com/user-attachments/assets/08eacde9-c028-41d1-9029-fe4d279e1470" />
face-detection-output.png


## ERROR Encountered 
My Curl command threw a 403 error. The request couldn't reach the Cloud Vision API
<img width="1852" height="310" alt="Screenshot 2026-09-28 100021" src="https://github.com/user-attachments/assets/f7aa2504-efe8-480b-aef9-88df1481ad1f" />

## Why

I forgot to run the export command to save the API key on Cloudshell 


## The Fix

Run the export command and the curl command again 

```bash
export API_KEY=<YOUR_API_KEY>
```


## Security notes

- **Never commit your API key.** Keep it in an environment variable, and add any `.env` file to `.gitignore`.
- Restrict the key to the Cloud Vision API only.
- Making a bucket object public exposes it to anyone with the link. Use non-sensitive sample images, and remove public access or delete the bucket when finished.
- Delete or rotate the API key after the project.

## What I learned

- How to structure and send REST requests to a cloud vision service
- How Cloud Storage links images to API calls
- How to interpret confidence scores and emotion likelihoods

## Ideas for next steps

- Add **text detection (OCR)**, logo detection, and SafeSearch, which the Vision API also supports
- Wrap the `cURL` calls in a Python script that processes a folder of images in batch
- Store results in BigQuery for analysis
- Compare API output with a custom model (for example, a CNN trained on a defect dataset) for inspection use cases

## Acknowledgements

Based on the Google Cloud Skills Boost lab *Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API*. Sample images are provided by the lab and are not included in this repository.

## Technologies

Google Cloud Platform, Cloud Storage, Cloud Vision API, REST, JSON, cURL, Cloud Shell
