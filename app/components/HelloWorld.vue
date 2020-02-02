<!-- Blog Post -->
<!-- https://timleland.com/compress-and-upload-image-using-nativescript -->
<template>
    <Page class="page">
        <ActionBar class="action-bar">
            <Label class="action-bar-title" text="Upload"></Label>
        </ActionBar>
        <StackLayout>
            <Button text="Select Picture" @tap="selectPics"
                class="-primary m-b-10" />
            <Image :src="image" stretch="aspectFill" width="100%"
                height="300"></Image>
        </StackLayout>
    </Page>
</template>

<script>
    import * as imagepicker from "nativescript-imagepicker";
    const imageSourceModule = require("tns-core-modules/image-source");
    const fs = require("file-system");
    const bgHttp = require("nativescript-background-http");
    const fsModule = require("tns-core-modules/file-system");
    const imageAssetModule = require(
        "tns-core-modules/image-asset/image-asset");
    export default {
        data() {
            return {
                image: null
            };
        },
        methods: {
            selectPics() {
                var self = this;
                let context = imagepicker.create({
                    mediaType: "Image",
                    mode: "single"
                });

                context
                    .authorize()
                    .then(() => {
                        return context.present();
                    })
                    .then(selection => {
                        selection.forEach(element => {
                            element.options.height = 100;
                            element.options.keepAspectRatio =
                            true;
                            self.image = element;
                            self.resizeImage(element).then(
                            path => {
                                self.uploadImage(path);
                            });
                        });
                    });
            },
            resizeImage(imageAsset) {
                var self = this;
                return imageSourceModule.ImageSource.fromAsset(imageAsset)
                    .then(
                        imageSource => {
                            const b64 = imageSource.toBase64String("jpg");
                            const fileSize = b64.replace(/\=/g, "").length *
                                0.75;
                            var compressionValue = 100;
                            if (fileSize.length > 5) {
                                compressionValue = 5;
                            }

                            const folder = fs.knownFolders.documents().path;
                            const fileName = "test.jpg";
                            const path = fs.path.join(folder, fileName);
                            var saved = imageSource.saveToFile(
                                path,
                                "jpg",
                                compressionValue
                            );
                            return path;
                        }
                    );
            },
            uploadImage(path) {
                var self = this;

                let file = fs.File.fromPath(path);
                let currentFileNameBeingUploaded = file.path.substr(
                    file.path.lastIndexOf("/") + 1
                );
                let request = self.createNewRequest();
                request.description = "uploading image " + file.path;
                request.headers["File-Name"] = currentFileNameBeingUploaded;
                var bgHttpSession = bgHttp.session(path);

                let task = bgHttpSession.uploadFile(path, request);

                task.on("error", function(event) {
                    alert({
                        title: "Whoops",
                        okButtonText: "OK",
                        message: "Something went wrong."
                    });
                });

                task.on("complete", function(event) {
                    alert({
                        title: "Complete",
                        okButtonText: "OK",
                        message: "Image uploaded."
                    });
                });
            },
            createNewRequest(uploadUrl) {
                let request = {
                    url: "https://jsonplaceholder.typicode.com/posts",
                    method: "POST",
                    headers: {
                        "Content-Type": "application/octet-stream"
                    },
                    description: "Uploading image..."
                };

                return request;
            }
        }
    };
</script>