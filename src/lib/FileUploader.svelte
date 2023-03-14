<script lang="ts">
    import * as tus from "tus-js-client";

    let success = false;
    let fail = false;

    let started = false;
    let paused = false;

    let progress = 0;
    let source: File | null = null;
    let uploader: tus.Upload | null = null;
    function change(this: HTMLInputElement, event: any) {
        const file = this.files.item(0);
        console.log(file);

        if (file) {
            source = file;
            uploader = new tus.Upload(source, {
                endpoint: "http://127.0.0.1:5002/api/v1/media/video/",
                retryDelays: [0, 3000, 5000, 10000, 20000],
                metadata: {
                    filename: source.name,
                    filetype: source.type,
                },
                onError: function (error) {
                    fail = true;
                    started = false;
                    paused = false;
                    console.error(error);
                },
                onProgress: function (bytesUploaded, bytesTotal) {
                    progress = (bytesUploaded / bytesTotal) * 100;
                    progress /= 10;
                    var percentage = (
                        (bytesUploaded / bytesTotal) *
                        100
                    ).toFixed(2);
                    console.log(bytesUploaded, bytesTotal, percentage + "%");
                },
                onSuccess: function () {
                    success = true;
                    fail = false;
                    started = false;
                    paused = false;

                    // use this api to get upload data `uploader`
                    console.log(
                        "Download %s from %s",
                        uploader.file,
                        uploader.url
                    );
                },
            });
        }
    }
    function submit(this: HTMLFormElement, event: any) {
        if (!source && !uploader) return;
        success = false;
        fail = false;

        start();
    }
    function start() {
        started = true;
        paused = false;
        uploader.findPreviousUploads().then(function (previousUploads) {
            if (previousUploads.length) {
                uploader.resumeFromPreviousUpload(previousUploads[0]);
            }
            uploader.start();
        });
    }
    function pause() {
        started = false;
        paused = true;
        uploader.abort();
    }
    function cancel() {
        started = false;
        paused = false;
        uploader.abort(true);
        progress = 0;
    }
</script>

<form method="post" on:submit|preventDefault={submit}>
    <h1>File Uploader</h1>
    <label for="uploader">Video</label>
    <input type="file" name="video" id="uploader" on:change={change} />
    {#if progress}
        <progress value={progress} />
    {/if}
    {#if success}
        <div>Success</div>
    {/if}
    {#if fail}
        <div>Fail</div>
    {/if}
    <button disabled={started}>Start</button>
    <button disabled={paused} on:click={pause} type="button">Pause</button>
    <button on:click={cancel} type="button">Cancel</button>
</form>

<style>
    form {
        display: grid;
        gap: 8px;
    }
    h1 {
        font-size: 32px;
        font-weight: 500;
    }
    input {
        padding: 20px;
        border-radius: 8px;
        background: #f1f1f1;
        color: black;
    }
    progress {
        width: 100%;
    }
</style>
