<script>
  import { flip } from "svelte/animate";
  import { quintOut } from "svelte/easing";
  import { crossfade } from "svelte/transition";

  const [send, receive] = crossfade({
    duration: d => Math.sqrt(d * 800),

    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === "none" ? "" : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
      };
    }
  });
  let images = [
    { src: "/assets/haru.jpg", alt: "Ha Ru Restaurant", id: "HaRuImage" },
    {
      src: "/assets/chickandbeers.png",
      alt: "Chick & Beers",
      id: "chickImage"
    },
    {
      src: "/assets/treestone.png",
      alt: "Treestone BBQ",
      id: "imagtreestoneImage"
    },
    { src: "/assets/han.jpg", alt: "Han Bar", id: "hanImage" },
    { src: "/assets/ktown.png", alt: "K-Town Buffet", id: "kTownImage" },
    { src: "/assets/yami.jpg", alt: "Yami Restaurant", id: "yamiImage" },
    { src: "/assets/jeonju.png", alt: "Jeonju Restaurant", id: "jeonjuImage" }
  ];

  let imageGroups = [
    { name: "🤤", items: [] },
    { name: "🙂", items: [] },
    { name: "😐", items: [] },
    { name: "🙁", items: [] },
    { name: "🤢", items: [] },
    { items: images }
  ];
  const dragstart = (ev, group, item) => {
    ev.dataTransfer.setData(
      "text/plain",
      JSON.stringify({ group: group, item: item })
    );
  };
  const dragover = ev => {
    ev.preventDefault();
    ev.dataTransfer.dropEffect = "move";
  };
  const drop = (ev, new_g) => {
    ev.preventDefault();
    var data = JSON.parse(ev.dataTransfer.getData("text/plain"));
    const i = +data.item;
    const old_g = +data.group;
    const xPositions = [];
    imageGroups[new_g].items.forEach((image, index) => {
      if (!(new_g === old_g && index === i)) {
        const el = document.getElementById(image.id);
        const rect = el.getBoundingClientRect();
        xPositions.push(rect.left + 50);
      }
    });
    const lowerXPositions = [...xPositions.filter(x => x < ev.x)];
    const imageGroupCopy = [...imageGroups];
    const item = imageGroupCopy[old_g].items.splice(i, 1)[0];
    imageGroupCopy[new_g].items.splice(lowerXPositions.length, 0, item);
    imageGroups = [...imageGroupCopy];
  };

  let addRowClass = (row, index) => {
    if (!row.name) {
      return "imageTray";
    }
    if (index % 2 === 0) {
      return "even";
    }
    return "odd";
  };
</script>

<style type="text/scss" lang="scss">
  @import "variables";
  span {
    display: flex;
    align-items: center;
    border-right: 2px solid $shadeLight;
  }

  div {
    width: 100%;
    display: flex;
    justify-content: center;
    flex-direction: row;
    min-height: 82px;
  }

  h2 {
    width: 100px;
    text-align: center;
    font-size: 2.4em;
    margin: 0;
  }

  img {
    width: 80px;
    height: 80px;
    -webkit-box-shadow: 2px 2px 8px 0px rgba(0, 0, 0, 0.4);
    -moz-box-shadow: 2px 2px 8px 0px rgba(0, 0, 0, 0.4);
    box-shadow: 2px 2px 8px 0px rgba(0, 0, 0, 0.4);
  }

  .imageRow {
    display: grid;
    padding: 0 0.4em;
    grid-column-gap: 0.4em;
    grid-row-gap: 0.2em;
    align-items: center;
    grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
    min-height: 100px;

    &.imageTray {
      padding: 1em 0;
    }
  }

  .odd {
    background-color: adjust-color($accentLight, $lightness: 25%);
  }

  .even {
    background-color: adjust-color($accentLight, $lightness: 17%);
  }
</style>

{#each imageGroups as imageGroup, i}
  <div>
    {#if imageGroup.name}
      <span class={i % 2 === 0 ? 'even' : 'odd'}>
        <h2>{imageGroup.name}</h2>
      </span>
    {/if}
    <div
      class={`imageRow ${addRowClass(imageGroup, i)}`}
      on:drop={event => drop(event, i)}
      on:dragover={dragover}>
      {#each imageGroup.items as image, j (image.id)}
        <img
          id={image.id}
          src={image.src}
          alt={image.alt}
          title={image.alt}
          on:dragstart={event => dragstart(event, i, j)}
          in:receive={{ key: image.id }}
          out:send={{ key: image.id }}
          animate:flip={{ duration: 800 }} />
      {/each}
    </div>
  </div>
{/each}
