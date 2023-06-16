<script>
	import { onMount } from 'svelte';

	let videoSource = null;
	let loading = false;

	const obtenerVideoCamara = async () => {
		try {
			loading = true;
			const stream = await navigator.mediaDevices.getUserMedia({ video: true });
			videoSource.srcObject = stream;
			videoSource.play();
			loading = false;
		} catch (error) {
			console.log(error);
		}
	};

	const capturarImagen = () => {
		const canvas = document.createElement('canvas');
		const context = canvas.getContext('2d');
		canvas.width = videoSource.videoWidth;
		canvas.height = videoSource.videoHeight;
		context.translate(canvas.width, 0); // Mirror the canvas horizontally
		context.scale(-1, 1); // Mirror the canvas vertically
		context.drawImage(videoSource, 0, 0, canvas.width, canvas.height);

		const dataUrl = canvas.toDataURL('image/png');
		const link = document.createElement('a');
		link.href = dataUrl;
		link.download = 'captured-image.png';
		link.click();
	};

	onMount(obtenerVideoCamara); // Call the function when the component is mounted


	const API_TOKEN = "608f92e0f5e84eac92eba263b3bef3bc";

	// Detecting gender and age
	function detect(image, callback){
    var myHeaders = new Headers();
    myHeaders.append("token", API_TOKEN);

    var formdata = new FormData();    

    if ((typeof image == "string") && (image.indexOf("https://") == 0))
        formdata.append("photo", image);
    else
        formdata.append("photo", image, "file");

    var requestOptions = {
      method: 'POST',
      headers: myHeaders,
      body: formdata,
      redirect: 'follow'
    };

    fetch("https://api.luxand.cloud/photo/detect", requestOptions)
      .then(response => response.json())
      .then(result => callback(result))
      .catch(error => console.log('error', error)); 
}

	function detect_faces(){
		// @ts-ignore
		var photo = document.getElementsByName("photo")[0].files[0];
		
		detect(photo, function(result){
			// @ts-ignore
			document.getElementById("detect-result").innerHTML = JSON.stringify(result, null, 8)
				.replace(/\ /g, "&nbsp;")
				.replace(/\n/g, "<br/>")

			// @ts-ignore
			document.getElementById("detect-result")["style"]["display"] = "block"
		});
	}
</script>

<div>
	{#if loading}
		<h1>LOADING</h1>
	{/if}
	<!-- svelte-ignore a11y-media-has-caption -->
	<video class="mirrored-video" bind:this={videoSource} />
	<button on:click={capturarImagen}>Capture and Download</button>

	<br/><br/>

	<div>
        <h2>Gender and age detection demo</h2>
        
        <div style="padding-bottom: 20px;">Please choose the photo for detection</div>
        <input type="file" name="photo" on:change={detect_faces}/>
        <div id="detect-result" style="display: none; font-family: monospace; background-color: #606a94; color: white; margin-top: 20px; max-width: 600px; padding: 10px;"></div>
    </div>
</div>

<style>
	.mirrored-video {
		transform: scaleX(-1);
	}
</style>
