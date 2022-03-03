<script>
  import { onMount } from "svelte";
  import mapboxgl from "mapbox-gl";

  let container;

  onMount(() => {
    let child = document.createElement("div");
    child.setAttribute("id", "map");
    child.style.height = "500px";
    container.appendChild(child);

    mapboxgl.accessToken =
      "pk.eyJ1IjoiZGlsbG9uLWRyb25lIiwiYSI6ImNrN3J6ZzlmdzBkdXgzZW5yeGVtYnVqYjkifQ.6px95nykLjyGtFlBdbsZ7w";
    const map = new mapboxgl.Map({
      container: "map",
      zoom: 11.53,
      center: [138.7189, 35.1691],
      pitch: 76,
      bearing: -177.2,
      style: "mapbox://styles/mapbox/satellite-v9",
      interactive: false,
    });

    // add terrain and sky layers once the style has loaded
    map.on("load", () => {
      map.addSource("mapbox-dem", {
        type: "raster-dem",
        url: "mapbox://mapbox.mapbox-terrain-dem-v1",
        tileSize: 512,
        maxzoom: 14,
      });
      map.setTerrain({ source: "mapbox-dem", exaggeration: 1.5 });

      map.addLayer({
        id: "sky",
        type: "sky",
        paint: {
          "sky-type": "atmosphere",
          "sky-atmosphere-sun": [0.0, 90.0],
          "sky-atmosphere-sun-intensity": 15,
        },
      });
    });

    function updateCameraPosition(position, altitude, target) {
      const camera = map.getFreeCameraOptions();

      camera.position = mapboxgl.MercatorCoordinate.fromLngLat(position, altitude);
      camera.lookAtPoint(target);

      map.setFreeCameraOptions(camera);
    }

    let animationIndex = 0;
    let animationTime = 0.0;

    // wait for the terrain and sky to load before starting animations
    map.once("idle", () => {
      // linearly interpolate between two altitudes/positions based on time
      const lerp = (a, b, t) => {
        if (Array.isArray(a) && Array.isArray(b)) {
          const result = [];
          for (let i = 0; i < Math.min(a.length, b.length); i++)
            result[i] = a[i] * (1.0 - t) + b[i] * t;
          return result;
        } else {
          return a * (1.0 - t) + b * t;
        }
      };

      const animations = [
        {
          duration: 20000.0,
          animate: (phase) => {
            const start = [138.73375, 35.41914];
            const end = [138.72649, 35.33974];
            const alt = [7000.0, 6000.0];

            // interpolate camera position while keeping focus on a target lat/lng
            const position = lerp(start, end, phase);
            const altitude = lerp(alt[0], alt[1], phase);
            const target = [138.73036, 35.36197];

            updateCameraPosition(position, altitude, target);
          },
        },
        {
          duration: 15000.0,
          animate: (phase) => {
            const start = [138.72649, 35.33974];
            const end = [138.72623, 35.31977];
            const alt = [6000.0, 6000.0];
            const target1 = [138.73036, 35.36197];
            const target2 = [138.74831, 35.34784];

            // interpolate both the camera position and target
            const position = lerp(start, end, phase);
            const altitude = lerp(alt[0], alt[1], phase);
            const target = lerp(target1, target2, phase);

            updateCameraPosition(position, altitude, target);
          },
        },
        {
          duration: 15000.0,
          animate: (phase) => {
            // create easing function for the animation
            const easeInOutQuad = (t) => {
              return t < 0.5 ? 2.0 * t * t : (4.0 - 2.0 * t) * t - 1.0;
            };
            const start = [138.72623, 35.31977];
            const end = [138.73375, 35.41914];
            const alt = [6000.0, 7000.0];
            const target1 = [138.74831, 35.34784];
            const target2 = [138.73036, 35.36197];

            // interpolate both the camera position and target
            const position = lerp(start, end, easeInOutQuad(phase));
            const altitude = lerp(alt[0], alt[1], phase);
            const target = lerp(target1, target2, phase);

            updateCameraPosition(position, altitude, target);
          },
        },
      ];

      let lastTime = 0.0;
      function frame(time) {
        animationIndex %= animations.length;
        const current = animations[animationIndex];

        if (animationTime < current.duration) {
          // Normalize the duration between 0 and 1 to interpolate the animation
          const phase = animationTime / current.duration;
          current.animate(phase);
        }

        // Elasped time since last frame, in milliseconds
        const elapsed = time - lastTime;
        animationTime += elapsed;
        lastTime = time;

        if (animationTime > current.duration) {
          animationIndex++;
          animationTime = 0.0;
        }

        window.requestAnimationFrame(frame);
      }

      window.requestAnimationFrame(frame);
    });

    // mapboxgl.accessToken =
    //       "pk.eyJ1IjoiZGlsbG9uLWRyb25lIiwiYSI6ImNrN3J6ZzlmdzBkdXgzZW5yeGVtYnVqYjkifQ.6px95nykLjyGtFlBdbsZ7w";
    //     const map = new mapboxgl.Map({
    //       style: "mapbox://styles/mapbox/light-v10",
    //       center: [-60.1738812595305, 46.115967460228674],
    //       zoom: 15.5,
    //       pitch: 45,
    //       bearing: -17.6,
    //       container: "map",
    //       antialias: true,
    //     });

    //     map.on("load", () => {
    //       // Insert the layer beneath any symbol layer.
    //       const layers = map.getStyle().layers;
    //       const labelLayerId = layers.find(
    //         (layer) => layer.type === "symbol" && layer.layout["text-field"]
    //       ).id;

    //       // The 'building' layer in the Mapbox Streets
    //       // vector tileset contains building height data
    //       // from OpenStreetMap.
    //       map.addLayer(
    //         {
    //           id: "add-3d-buildings",
    //           source: "composite",
    //           "source-layer": "building",
    //           filter: ["==", "extrude", "true"],
    //           type: "fill-extrusion",
    //           minzoom: 15,
    //           paint: {
    //             "fill-extrusion-color": "#aaa",

    //             // Use an 'interpolate' expression to
    //             // add a smooth transition effect to
    //             // the buildings as the user zooms in.
    //             "fill-extrusion-height": [
    //               "interpolate",
    //               ["linear"],
    //               ["zoom"],
    //               15,
    //               0,
    //               15.05,
    //               ["get", "height"],
    //             ],
    //             "fill-extrusion-base": [
    //               "interpolate",
    //               ["linear"],
    //               ["zoom"],
    //               15,
    //               0,
    //               15.05,
    //               ["get", "min_height"],
    //             ],
    //             "fill-extrusion-opacity": 0.6,
    //           },
    //         },
    //         labelLayerId
    //       );
  });
</script>

<div bind:this={container} />
