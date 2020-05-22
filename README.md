# Host Your Own Image Server based on Google Drive

[![Go Report Card](https://goreportcard.com/badge/github.com/selcukusta/gdrive-image-server)](https://goreportcard.com/report/github.com/selcukusta/gdrive-image-server)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit&logoColor=white)](https://github.com/selcukusta/gdrive-image-server)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/selcukusta/gdrive-image-server/blob/master/LICENSE)

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=selcukusta_gdrive-image-server&metric=alert_status)](https://sonarcloud.io/dashboard?id=selcukusta_gdrive-image-server)
[![Technical Debt](https://sonarcloud.io/api/project_badges/measure?project=selcukusta_gdrive-image-server&metric=sqale_index)](https://sonarcloud.io/dashboard?id=selcukusta_gdrive-image-server)
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=selcukusta_gdrive-image-server&metric=bugs)](https://sonarcloud.io/dashboard?id=selcukusta_gdrive-image-server)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=selcukusta_gdrive-image-server&metric=code_smells)](https://sonarcloud.io/dashboard?id=selcukusta_gdrive-image-server)
[![Duplicated Lines (%)](https://sonarcloud.io/api/project_badges/measure?project=selcukusta_gdrive-image-server&metric=duplicated_lines_density)](https://sonarcloud.io/dashboard?id=selcukusta_gdrive-image-server)

Turn your Google Drive folder to the image hosting service!

Now, MongoDB GridFS is fully supported!

## Getting Started

### Google Drive

You need to have a Google account to set up the project.

#### Setup steps for using Google Drive API

Firstly, create a new project from [Google Developer Console](https://console.developers.google.com/). Go to the **Credentials** menu and create a new **Service Accounts** credential. It might be named as `[PROJECT_ALIAS]-xxxx-xxxxxxx.json`.

Download it and rename as `gcloud-image-server-cred.json`. Put the file to the **root** folder (_it will be used for building Docker image_).

Copy your service account mail address (_it will be used for sharing your images with the project_).

#### Setup steps for host the images

Go to your Drive page and create a folder, ie. `image-server`.

Share the folder with your service account (_was copied before_). Sharing rule will be applied to the all sub-items in the folder.

As a last step, upload any image (mime should be `image/jpeg` or `image/png`) to the folder and get the ID.

_NOTE: `ID` is not showing anywhere in the portal, it sucks! To catch it, right click your item and select `Get Shareable Link`. Copy the last part of it, and toggle off the sharable link feature._

### MongoDB

Before running the application set these environment variables (or use `Docker` image, run with `docker container run -d --name mongodb-instance -p 27017:27017 mongo:3.6.18-xenial` and leave them default):

| Name                   | Type     | Default Value               |
| :--------------------- | :------- | :-------------------------- |
| `MONGO_CONNECTION_STR` | `string` | _mongodb://127.0.0.1:27017_ |
| `MONGO_MAX_POOL_SIZE`  | `uint64` | _5_                         |

Run `go run cmd/mongo-seed/main.go` command and create 3 sample record on the DB such as;

| \_id                     | chunkSize | filename    | length | metadata                       | uploadDate               |
| :----------------------- | :-------- | :---------- | :----- | :----------------------------- | :----------------------- |
| 5ec684803dd893bb72ead932 | 261120    | image-3.jpg | 710863 | {"Content-Type": "image/jpeg"} | 2020-05-21T13:39:12.585Z |
| 5ec684803dd893bb72ead931 | 261120    | image-2.jpg | 516218 | {"Content-Type": "image/jpeg"} | 2020-05-21T13:39:12.603Z |
| 5ec684803dd893bb72ead930 | 261120    | image-1.jpg | 379373 | {"Content-Type": "image/jpeg"} | 2020-05-21T13:39:12.617Z |

## Running

- `docker-compose up`

- Go to your favorite browser

- Surf to the URLs such as below.

Two containers will be bringing up. The first one is your image server application. It was written with purely **[the Go programming language](https://golang.org/)**.

Another one is **[Varnish HTTP Cache](https://varnish-cache.org/)**. It will automatically cache the output of the response for 14 days.

## Usage

/i/ endpoint is used for image operations, has two different usages:

### Google Drive

```
/i/gdrive/{quality:range(0,100)}/{w:range(0,5000)}x{h:range(0,5000)}/{options:opt}/{*id}

/i/gdrive/{quality:range(0,100)}/{w:range(0,5000)}x{h:range(0,5000)}/{*id}
```

### MongoDB

```
/i/gridfs/{quality:range(0,100)}/{w:range(0,5000)}x{h:range(0,5000)}/{options:opt}/{*id}

/i/gridfs/{quality:range(0,100)}/{w:range(0,5000)}x{h:range(0,5000)}/{*id}
```

## Options

| Option | Description                                                                                                                                                                                                                         |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `t`    | Make this image into a thumbnail. This method modifies the image to contain a thumbnail version of itself, no larger than the given size. This method calculates an appropriate thumbnail size to preserve the aspect of the image. |
| `g`    | This will convert the given image into a grayscale image.                                                                                                                                                                           |
| `s`    | This will crop the given image smartly with using [this](https://pkg.go.dev/github.com/muesli/smartcrop) package.                                                                                                                   |

## Samples

- **Original size:** `http://127.0.0.1:8080/i/gdrive/100/0x0/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/100/0x0/[MONGODB_OBJECT_ID]`

![1](assets/1.png)

- **Resize with aspect ratio:** `http://127.0.0.1:8080/i/gdrive/100/500x0/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/100/500x0/[MONGODB_OBJECT_ID]`

![2](assets/2.png)

- **Less quality:** `http://127.0.0.1:8080/i/gdrive/1/0x0/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/1/0x0/[MONGODB_OBJECT_ID]`

![3](assets/3.png)

- **Resize without aspect ratio:** `http://127.0.0.1:8080/i/gdrive/100/1600x600/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/100/1600x600/[MONGODB_OBJECT_ID]`

![4](assets/4.png)

- **Resize with smart crop:** `http://127.0.0.1:8080/i/gdrive/100/1600x600/s/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/100/1600x600/s/[MONGODB_OBJECT_ID]`

![5](assets/5.png)

- **Create thumbnail with aspect ratio:** `http://127.0.0.1:8080/i/gdrive/100/0x300/t/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/100/0x300/t/[MONGODB_OBJECT_ID]`

![6](assets/6.png)

- **Grayscale:** `http://127.0.0.1:8080/i/gdrive/100/900x0/g/[YOUR_FILE_ID]` or `http://127.0.0.1:8080/i/gridfs/100/900x0/g/[MONGODB_OBJECT_ID]`

![7](assets/7.png)

## LICENSE

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
