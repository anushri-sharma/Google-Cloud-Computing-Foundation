GSP315

# [Set Up an App Dev Environment on Google Cloud: Challenge Lab](https://www.skills.google/paths/36/course_templates/637/labs/592550)

### [Setup](https://www.youtube.com/watch?v=X56AFb7JvAs)

# Introduction
In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the course to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!

This lab is recommended for students who have enrolled in the Set Up an App Dev Environment on Google Cloud skill badge. Are you ready for the challenge?


## Challenge scenario

You are just starting your junior cloud engineer role with Jooli inc. So far you have been helping teams create and manage Google Cloud resources.

You are expected to have the skills and knowledge for these tasks, so don’t expect step-by-step guides.

### Your challenge
You are asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called Memories. You have been asked to assist the Memories team with initial configuration for their application development environment.

You receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Run Function you create.
- Create a Cloud Run Function.
- Remove the previous cloud engineer’s access from the memories project.

>>> Some Jooli Inc. standards you should follow:

- Create all resources in the REGION region and ZONE zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named kraken-webserver1
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use e2-micro for small Linux VMs and e2-medium for Windows or other applications such as Kubernetes nodes.


Each task is described in detail below, good luck!

# Task 1. Create a bucket
> You need to create a bucket called Bucket Name for the storage of the photographs. Ensure the resource is created in the REGION region and ZONE zone.

## Task 1 Steps: [Create a Cloud Storage Bucket](https://github.com/anushri-sharma/Google-Cloud-Computing-Foundations/blob/77e60fed047f2620fa5a84d5e9d3054186243eee/Set%20Up%20an%20App%20Dev%20Environment%20on%20Google%20Cloud/1.%20Cloud%20Storage%3A%20Qwik%20Start%20-%20Google%20Cloud%20Console.md)

1. Open the Google Cloud Console using **Username 1** provided in the lab.
2. In the search bar, search for **Cloud Storage Buckets**.
3. Open the first result in a new tab.
4. Click **Create**.
5. Copy the bucket name from the lab instructions.
6. Paste it into the **Bucket Name** field.
7. Click **Continue**.
8. Select the required **Region** (example: `us-central1`).
9. Keep the default settings.
10. Click **Continue** → **Create**.
11. Click **Confirm** if prompted.
12. Wait until the bucket is created successfully.
13. Return to the lab page and click **Check My Progress** for Task 1.

---
# Task 2. Create a Pub/Sub topic

> Create a Pub/Sub topic called Topic Name for the Cloud Run Function to send messages.

# Task 2 Steps: [Create a Pub/Sub Topic](https://github.com/anushri-sharma/Google-Cloud-Computing-Foundations/blob/77e60fed047f2620fa5a84d5e9d3054186243eee/Set%20Up%20an%20App%20Dev%20Environment%20on%20Google%20Cloud/7.%20Pub%5CSub%3A%20Qwik%20Start%20-%20Console.md)

1. Search for **Pub/Sub** in the Google Cloud Console.
2. Open the first result in a new tab.
3. Click **Create Topic**.
4. Copy the Topic ID from the lab instructions.
5. Paste it into the **Topic ID** field.
6. Click **Create**.
7. Wait for the success message:

   * “New topic and subscription created successfully.”
8. Return to the lab page.
9. Click **Check My Progress** for Task 2.

---


# Task 3. [Create the thumbnail Cloud Run Function](https://github.com/anushri-sharma/Google-Cloud-Computing-Foundations/blob/main/Set%20Up%20an%20App%20Dev%20Environment%20on%20Google%20Cloud/5.%20Cloud%20Run%20Functions%3A%20Qwik%20Start%20-%20Console.md)

> Create the function

Create a Cloud Run Function Cloud Run Function Name that will to create a thumbnail from an image added to the Bucket Name bucket.

Ensure the Cloud Run Function is using the Cloud Run function environment (which is 2nd generation). Ensure the resource is created in the REGION region and ZONE zone.

> 1. Create a Cloud Run Function (2nd generation) called Cloud Run Function Name using Node.js 22.

>> Note: The Cloud Run Function is required to execute every time an object is created in the bucket created in Task 1. During the process, Cloud Run Function may request permission to enable APIs or request permission to grant roles to service accounts. Please enable each of the required APIs and grant roles as requested.

> 2. Make sure you set the Entry point (Function to execute) to Cloud Run Function Name and Trigger to Cloud Storage.

> 3. Add the following code to the index.js:

```
const functions = require('@google-cloud/functions-framework');
const { Storage } = require('@google-cloud/storage');
const { PubSub } = require('@google-cloud/pubsub');
const sharp = require('sharp');

functions.cloudEvent('', async cloudEvent => {
  const event = cloudEvent.data;

  console.log(`Event: ${JSON.stringify(event)}`);
  console.log(`Hello ${event.bucket}`);

  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64";
  const bucket = new Storage().bucket(bucketName);
  const topicName = "";
  const pubsub = new PubSub();

  if (fileName.search("64x64_thumbnail") === -1) {
    // doesn't have a thumbnail, get the filename extension
    const filename_split = fileName.split('.');
    const filename_ext = filename_split[filename_split.length - 1].toLowerCase();
    const filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length - 1); // fix sub string to remove the dot

    if (filename_ext === 'png' || filename_ext === 'jpg' || filename_ext === 'jpeg') {
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      const newFilename = `${filename_without_ext}_64x64_thumbnail.${filename_ext}`;
      const gcsNewObject = bucket.file(newFilename);

      try {
        const [buffer] = await gcsObject.download();
        const resizedBuffer = await sharp(buffer)
          .resize(64, 64, {
            fit: 'inside',
            withoutEnlargement: true,
          })
          .toFormat(filename_ext)
          .toBuffer();

        await gcsNewObject.save(resizedBuffer, {
          metadata: {
            contentType: `image/${filename_ext}`,
          },
        });

        console.log(`Success: ${fileName} → ${newFilename}`);

        await pubsub
          .topic(topicName)
          .publishMessage({ data: Buffer.from(newFilename) });

        console.log(`Message published to ${topicName}`);
      } catch (err) {
        console.error(`Error: ${err}`);
      }
    } else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  } else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
});
```
> 4. Add the following code to the package.json:

```
{
 "name": "thumbnails",
 "version": "1.0.0",
 "description": "Create Thumbnail of uploaded image",
 "scripts": {
   "start": "node index.js"
 },
 "dependencies": {
   "@google-cloud/functions-framework": "^3.0.0",
   "@google-cloud/pubsub": "^2.0.0",
   "@google-cloud/storage": "^6.11.0",
   "sharp": "^0.32.1"
 },
 "devDependencies": {},
 "engines": {
   "node": ">=4.3.2"
 }
}
```

>> Note: If you get a permission denied error stating it may take a few minutes before all necessary permissions are propagated to the Service Agent, wait a few minutes and try again. Ensure you have the appropriate roles (Eventarc Service Agent, Eventarc Event Receiver, Service Account Token Creator, and Pub/Sub Publisher) assigned to the correct service accounts.


## Test the function

- Upload a PNG or JPG image of your choice to the Bucket Name bucket.

>> Note: Alternatively, download this image https://storage.googleapis.com/cloud-training/gsp315/map.jpg to your machine. Then, upload it to the bucket.

You will see a thumbnail image appear shortly afterwards (use REFRESH on the bucket details page).

After you upload the image file, you can click to check your progress below. You do not need to wait for the thumbnail image to be created.

Optional: If the function deployed successfully and you do not see the thumbnail image in the bucket, you can check that the Triggers tab displays the trigger information that you previously provided for the function, which may not have saved correctly if you previously encountered errors. If you do not see the Cloud Storage trigger in the Triggers tab of the function, you can recreate the trigger (see the documentation page titled Create a trigger for services), and then upload a new file again to test again (refresh the page after adding a new file).


# Task 3 Steps: [Create a Cloud Run Function](https://github.com/anushri-sharma/Google-Cloud-Computing-Foundations/blob/main/Set%20Up%20an%20App%20Dev%20Environment%20on%20Google%20Cloud/5.%20Cloud%20Run%20Functions%3A%20Qwik%20Start%20-%20Console.md)

1. Search for **Cloud Run** in the console.
2. Open the first result.
3. Click **Write a Function**.

## Configure Function

### Basic Settings

1. Copy the **Service Name** from the lab instructions.
2. Paste it into the **Service Name** field.
3. Ensure the region is correct (`us-central1` in most labs).

---

## Add Trigger

1. Click **Add Trigger**.
2. Select **Cloud Storage Trigger**.
3. If prompted, click **Enable**.
4. Click **Cancel** and open **Add Trigger** again if necessary.
5. Select **Cloud Storage Trigger** again.
6. Click **Browse** in the bucket field.
7. Select the bucket created in Task 1.
8. Click **Select**.
9. Click **Grant** permissions whenever prompted.
10. After permissions are granted successfully:

    * Click **Save Trigger**.

---

## Runtime Settings

1. Ensure:

   * Runtime = `Node.js 22`
2. Expand:

   * **Container, Volume, Networking, Security**
3. Under **Execution Environment**, select:

   * **Second Generation**
4. Click **Create**.

---

## Configure Source Code

### Function Entry Point

1. Copy the function entry point name from the lab instructions.
2. Replace the existing entry point with the copied name.

---

### Update `index.js`

1. Copy the code provided in the lab instructions.
2. Replace all existing code inside `index.js`.

---

### Update `package.json`

1. Open `package.json`.
2. Copy the package.json code from the lab instructions.
3. Replace the existing code completely.

---

## Deploy Function

1. Click **Save and Redeploy**.
2. Wait until all deployment steps show green checkmarks.

   * This may take 1–3 minutes.

---

## Test the Function

1. Open **Cloud Storage Buckets** again.
2. Open the bucket created earlier.
3. Click **Upload Files**.
4. Upload any `.png` or `.jpg` image.

   * You can even upload a screenshot.
5. Wait a few seconds.

---

## Verify Task

1. Return to the lab instructions page.
2. Click **Check My Progress** for Task 3.
3. You should receive a green checkmark.

---
# Task 4. Remove the previous cloud engineer
You will see that there are two users defined in the project.

- One is your account (Username 1 with the role of Owner).
- The other is the previous cloud engineer (Username 2 with the role of Viewer).

1. Remove the previous cloud engineer’s access from the project.


# Task 4 Steps: [Remove IAM Permissions](https://github.com/anushri-sharma/Google-Cloud-Computing-Foundations/blob/main/Set%20Up%20an%20App%20Dev%20Environment%20on%20Google%20Cloud/3.%20Cloud%20IAM%3A%20Qwik%20Start.md)

1. Open the **Navigation Menu**.
2. Go to:

   * **IAM & Admin → IAM**
3. Copy the second Gmail ID from the lab instructions.
4. Paste it into the filter/search box.
5. Select the matching user.
6. Click **Edit Principal**.
7. Click the **Delete** icon/remove role.
8. Click **Save**.
9. Return to the lab page.
10. Click **Check My Progress** multiple times if needed.
11. Wait for the green checkmark.

---

# Lab Completed Successfully

After all tasks show green checkmarks:

1. Click **End Lab**.
2. The skill badge/course progress will be updated automatically.
