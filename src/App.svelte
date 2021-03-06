<script>
  import SplashBanner from "./UI/SplashBanner.svelte";
  import Tab, { Icon, Label } from "@smui/tab";
  import TabBar from "@smui/tab-bar";
  import Button from "@smui/button";
  import { onMount, afterUpdate, onDestroy } from "svelte";
  import TreeComponent from "./components/TreeComponent.svelte";
  import CollapsibleTree from "./components/CollapsibleTree.svelte";
  import Footer from "./UI/Footer.svelte";
  import Nav from "./UI/Nav.svelte";
  import LargeTreeContainer from "./containers/LargeTreeContainer.svelte";
  import DataTable from "./components/DataTable.svelte";
  import { Cart } from "./stores/Cart.js";
  import Dropdown from "./UI/Dropdown.svelte";
  import CartIndicator from "./UI/CartIndicator.svelte";
  import CartView from "./containers/CartView.svelte";
  import HelpMenu from "./UI/HelpMenu.svelte";
  // import LikeButton from './UI/LikeButton.svelte';
  import BrowserView from './components/BrowserView.svelte';
  import Drawer from './UI/Drawer.svelte';

  const virusList = ["Ebola", "SARS", "MERS", "SARS-CoV-2"];
  const virusNameList = ["ebola", "sars", "mers", "ncov"];
  let DATA = {};
  let virusName = "ncov";

  function handleReferenceSelect(event) {
    if (event.detail === "SARS-CoV-2") {
      virusName = "ncov";
    } else {
      virusName = event.detail.toLowerCase();
    }
    Cart.addDataItems([]);
    keyedTabsActive = iconTabs[2];
  }

  const unsubscribe = Cart.subscribe(async store => {
    const { data } = store;
		// window.BROWSER_DATA.a = data;
  });
	
	onDestroy(() => {
		unsubscribe();
	});

  onMount(() => {
    virusNameList.forEach(reference => {
      let fileHandle = require(`./metadata/${reference}_all_skinny.json`);
      DATA[reference] = fileHandle.map((d, i) => {
        const {
          accession,
          organism,
          isolate,
          mol_type,
          strain,
          db_xref,
          collection_date,
          country,
          host
        } = d;
        let tmp = { _id: i + 1 };
        tmp.Accession = accession;
        tmp.Organism = organism[0] || "";
        tmp.Molecule_Type = mol_type !== undefined ? mol_type[0] : "N/A";
        tmp.Strain = strain !== undefined ? strain[0] : "N/A";
        tmp.Isolate = isolate !== undefined ? isolate[0] : "N/A";
        tmp.Collection_Date =
          collection_date !== undefined ? collection_date[0] : "N/A";
        tmp.Country = country !== undefined ? country[0] : "N/A";
        tmp.db_xref = db_xref !== undefined ? db_xref[0] : "N/A";
        tmp.Host = host !== undefined ? host[0] : "N/A";
        return tmp;
      });
    });

    const TRACKS = sessionStorage.getItem('tracks');
    if (TRACKS !== '') {
      let parsedTracks = JSON.parse(TRACKS);
      Cart.addDataItems(parsedTracks.filter(d => d.type === 'pairwise').map(d => d.metadata));
    }

    let referenceLS = sessionStorage.getItem('reference');
    let parsedReference = JSON.parse(referenceLS);
    if (parsedReference) {
      virusName = parsedReference;
    }
  });

  let iconTabs = [
    {
      k: 0,
      icon: "",
      label: ""
    },
    {
      k: 1,
      icon: "green",
      label: "Tree View"
    },
    {
      k: 2,
      icon: "",
      label: "Data"
    },
    {
      k: 3,
      icon: "shopping_cart",
      label: "Cart"
    },
    {
      k: 4,
      icon: "web",
      label: "Browser View"
    }
  ];
  let keyedTabsActive = iconTabs[0];

  let helpMenuItems = [
    {
      k: 0,
      icon: "slideshow",
      label: "Video tutorials",
      url: "https://youtu.be/cOv8W28GzwM"
    },
    {
      k: 1,
      icon: "description",
      label: "Documentation",
      url: "https://virusgateway.readthedocs.io/en/latest/index.html"
    },
    {
      k: 2,
      icon: "code",
      label: "Github",
      url: "https://github.com/debugpoint136/WashU-Virus-Genome-Browser"
    }
  ];
</script>

<style>

</style>

<!-- <div class="bg-right bg-cover" style="background-image:url('/images/bg.svg');"> -->
<div>
  <!-- <Nav/> -->
  
  <div class="w-full mx-auto flex justify-start">
    <div class="w-full xl:w-1/5 xl:flex xl:items-center lg:justify-start">
      <a
        class="flex ml-4 items-center text-indigo-400 no-underline
        hover:no-underline font-bold text-2xl"
        href="/">
        <svg
          class="h-8 fill-current text-indigo-600 pr-2"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 20 20">
          <path
            d="M10 20a10 10 0 1 1 0-20 10 10 0 0 1 0 20zm-5.6-4.29a9.95 9.95 0 0
            1 11.2 0 8 8 0 1 0-11.2 0zm6.12-7.64l3.02-3.02 1.41 1.41-3.02 3.02a2
            2 0 1 1-1.41-1.41z" />
        </svg>
        WashU Virus Genome Browser
      </a>
    </div>
    <div class="w-full xl:w-3/5 xl:flex lg:items-start">
      <div class="m-2">
        <HelpMenu items={helpMenuItems} />
      </div>
      <Drawer/>
      <div class="flex flex-col m-2">
        <Dropdown
          on:reference-select={handleReferenceSelect}
          names={virusList} />
        <Label>Reference</Label>
      </div>

      <TabBar
        tabs={iconTabs}
        let:tab
        key={tab => tab.k}
        bind:active={keyedTabsActive}>
        <Tab
          {tab}
          stacked={true}
          indicatorSpanOnlyContent={true}
          tabIndicator$transition="fade">
          <Icon class="material-icons">{tab.icon}</Icon>
          <Label>
            {#if tab.k === 3}
              <CartIndicator />
            {:else}{tab.label}{/if}
          </Label>
        </Tab>
      </TabBar>
    </div>
  </div>
  <div id="main-wrapper" class="mx-16">
    <div>
      {#if keyedTabsActive.k === 0}
        <div style="height: 800px;">
          <SplashBanner on:start={() => (keyedTabsActive = iconTabs[2])} />
        </div>
      {:else if keyedTabsActive.k === 1}
        <div style="height: 800px;" class="container">
          <LargeTreeContainer {virusName} DATA={DATA[virusName]} />
        </div>
      {:else if keyedTabsActive.k === 2}
        <div style="height: 800px;">
          <DataTable {virusName} DATA={DATA[virusName]} />
        </div>
      {:else if keyedTabsActive.k === 3}
        <div style="height: 800px;" class="container">
          <CartView />
          </div>
      {:else if keyedTabsActive.k === 4}
        <div></div>
      {/if}
    </div>
    <div class="w-full xl:w-1/5 lg:items-start"></div>
  </div>
  <div class:hidden={keyedTabsActive.k !== 4}>
    <BrowserView {virusName} active={keyedTabsActive.k}/>
  </div>

  <!-- <TreeComponent /> -->
  <!-- <CollapsibleTree/> -->

  <!-- TEST -->
  <!-- <SplashBanner /> -->
  <!-- <Footer class="mt-14"/> -->

</div>
