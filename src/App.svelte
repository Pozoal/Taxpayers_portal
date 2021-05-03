<script>
	import { onMount } from "svelte";
	import { flow, map, uniq, sortBy, identity, filter, find } from "lodash/fp";
	import PlayList from "./playlist.svelte";

	let config = {
		platform: "GRA",
		gettingStartVideoUrl: "",
		defaultRole: "",
		allowGlobalVideoSearch: true,
		allowRoleVideoSearch: true,
	};
	let isSideNavOpen = false;
	let switcherOpen = false;
	let allVideos = [];
	let roles = [];
	let activeRole = "";
	let roleVideos = [];
	let filteredRoleVideos = [];
	let roleSearchValue = "";
	let headerSearchValue = "";

	let searchData = [];
	let filteredSearchData = [];
	let searchActive = false;

	let activeVideo = { url: "", title: "", description: "" };

	onMount(async () => {
		//get the configuration
		const configResponse = await fetch("/config.json");
		const configData = await configResponse.json();
		config = configData;
		activeVideo = {
			url: config.gettingStartVideoUrl,
			title: "Getting Started",
			description: "",
		};
		//get the videos
		const response = await fetch("/videos.json");
		allVideos = await response.json();
		roles = flow(
			map((x) => x.role),
			uniq,
			sortBy(identity)
		)(allVideos);

		searchData = flow(
			map((x) => {
				return { text: x.title, description: x.role, ref: x };
			}),
			sortBy((x) => x.text)
		)(allVideos);
	});

	function activateRole(role) {
		isSideNavOpen = false;
		activeRole = role;
		roleVideos = flow(filter((x) => x.role === role))(allVideos);
		filteredRoleVideos = roleVideos;
		roleSearchValue = "";
	}

	function filterRoleVideos(roleSearchValue) {
		if (!roleSearchValue) {
			filteredRoleVideos = roleVideos;
			return;
		}
		else{
			filteredRoleVideos = allVideos.includes(roleSearchValue);
		return filteredRoleVideos;
		}
		
	}

	function playVideo(video) {
		document.getElementById("myVideo").autoplay = true;
		activeVideo = {
			url: video.url,
			title: video.title,
			description: video.role || activeRole,
		};

		
	}

	function playVideoFromSearch(e) {
		if (!e || !e.detail) return;
		var video = e.detail.selectedResult.ref;
		if (!video) {
			return;
		}
		playVideo(video);
	}

	let searchVid = "";

	let filteredVids = [];
	const searchVideos = () => {	
		return filteredVids = allVideos.filter(vids => {
			let videoTitle = vids.title.toLowerCase();
			return videoTitle.includes(searchVid.toLowerCase())
		});
	}

	$: filterRoleVideos(roleSearchValue);
	$: filteredSearchData = !!headerSearchValue
		? searchData.filter((x) => {
				return (
					x.text
						.toLowerCase()
						.includes(headerSearchValue.toLowerCase()) ||
					x.description
						.toLowerCase()
						.includes(headerSearchValue.toLowerCase())
				);
		  })
		: searchData;

	
</script>

<body>
	<div id="app">
		<nav
			class="navbar navbar-light sticky-top bg-light flex-md-nowrap px-4 shadow-sm">
			<div class="col-md-3 col-lg-2">
				<a class="navbar-brand d-flex" href="/index.html">
					<img
						src="images/gra_logo.png"
						height="30"
						alt=""
						loading="lazy" />
					<small class="align-self-center text-secondary">- TAXPAYERS PORTAL</small>
				</a>
			</div>

			
			<input
				class="form-control form-control-light border-info"
				type="text"	
				placeholder="Search video"
				bind:value={searchVid}
				on:input={searchVideos}
				aria-label="Search" />
		</nav>

		<div class="container-fluid pr-3">
			<div class="row">
				
				<div class="col-md-1 col-sm-1"></div>  
				<main
					role="main"
					class="d-lg-flex col-md-10 ml-sm-auto col-lg-10">
					<div class="col-lg px-3">
						{#if activeVideo && activeVideo.url}
							

							{#key activeVideo.url}
								<video
									id="myVideo"
									title="video"
									controls
									poster="images/poster.png"
									style="object-fit: cover;">
									<source
										src={activeVideo.url}
										type="video/mp4" />
									<track kind="captions" />
									Sorry, your browser does not support embeded
									videos
								</video>
								<div class="card my-3">
									<div class="card-body p-0">
										<div style="padding:1rem;">
											<div
												class="d-flex justify-content-between">
												<h6
													class="card-text text-primary text-uppercase text-weight-bold m-0">
													{activeVideo.title}
												</h6>
												<p class="text-muted m-0">
													Now Playing
												</p>
											</div>
										</div>
									</div>
								</div>
							{/key}
						{:else}
							<p>No video selected</p>	
						{/if}
					</div>
					
					
					<div class="playlist-cont container col-lg-4 pt-4">
						<h6 class="text-center text-uppercase text-muted">
							Current Playlist
						</h6>
						{#if searchVid && filteredVids.length === 0}
						<p>No results</p>
						{:else}						
								<div class="video-list border">
									<PlayList
										videos={allVideos}
										on:play={(e) => playVideo(e.detail)} />
								</div>

						
								
						{/if}
					</div>
				</main>
			</div>
		</div>
	</div>
</body>
