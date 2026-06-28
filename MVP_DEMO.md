# MVP Demo Evidence

Demo video evidence folder:

```text
https://drive.google.com/drive/folders/1tjFNgdLY_eRJ8PTQr1caMf6FQ1IobFcj?usp=sharing
```

## Included Demo Videos

1. Robot movement demo
   - Shows the physical robot moving in the real environment.
   - This is a hardware movement demo for the MVP.

2. Inspectra web app demo
   - Shows the web user flow for surface inspection reports.
   - Demonstrates reports saved from the robot/camera inspection flow.
   - Demonstrates uploading an image from the computer, running VLM defect
     recognition, and saving the report under the logged-in user account.

## End-to-End Flow Covered

```text
camera/manual image -> VLM surface inspection -> report artifacts ->
backend persistence -> user-owned report list/detail page -> human review
```

Important boundary:

```text
Camera images are used for inspection capture only.
They are not used to decide robot navigation in the MVP.
```
