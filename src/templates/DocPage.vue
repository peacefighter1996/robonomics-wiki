<template>
  <Layout>

  <div class="sidebarMobileToggle">
    <div class="layout__page flex-line">

      <NavIcon :section="'sidebarDocs'" :icon="'SideLeft'"/>
      <NavIcon :section="'sidebarContent'" :icon="'SideRight'"/>
      
    </div>
  </div>

	<div class="page">
		<div id="sidebarDocs" class="page__sidebar hiddenMobile">
      <SidebarDocs :items="items" />
      <Banner />
    </div>

  	<div class="page__content">
  		<VueRemarkContent />
      <!--<Banner :place="'content'" />-->

    
    
    <!--{{items}}
    <hr/>
    {{linksList}}

    <section class="page__next">
    
      <g-link v-if="previousPage" exact class="button button__secondary" :to="previousPage.link">
        &larr; {{ previousPage.title }}
      </g-link>
      <g-link v-if="nextPage" exact class="button button__secondary" :to="nextPage.link">
        {{ nextPage.title }} &rarr;
      </g-link>

    </section>-->

  	</div>

  	<div id="sidebarContent" class="page__sidebar hiddenMobile">
      <SidebarContent />
      <a :href="editLink" target="_blank" class="button button__secondary button__small">
        <IconGithub/>
        <span>Edit this page</span>
      </a>
    </div>

  </div>


  </Layout>
</template>

<style lang="scss">

  .page{
      display: grid;
      // grid-template-columns: 250px auto 200px;
      grid-template-columns: minmax(0,250px) minmax(0,auto) minmax(0,200px);
      gap: var(--space);
      align-items: start;

      h1 { font-weight: 500; }

      &__sidebar{
        word-break: break-word;

        overflow-y: auto;
        max-height: calc(100vh - 80px);
        scrollbar-width: none;  /* Firefox */
        -ms-overflow-style: none;  /* IE and Edge */
        &::-webkit-scrollbar { display: none; } /* Hide scrollbar for Chrome, Safari and Opera */
      }

      &__next {
        border-width: 1px 0 0;
        border-style: solid;
        border-color: var(--border-color);
        padding: calc(var(--space)/2) 0;

        display: grid;
        gap: calc(var(--space)/2);

        @media screen and (min-width:500px){
          grid-template-columns: 1fr 1fr;
        }

        text-transform: uppercase
      }

      @media screen and (min-width: 1080px){
        
        &__sidebar {
          position: sticky;
          top: 80px;
        }

      }
  }

  #sidebarDocs.page__sidebar {

    .menu {
      h4 {
        border-top: 1px solid var(--border-color);
        padding-top: 5px;

        &:not(:first-child) { margin-top: 20px; }
      }
	    padding-left: calc( var(--space) / 4);
    }

    .menu .menu:hover{
      border-color: var(--link-color);
    }
    
  }

  .sidebarMobileToggle{
    transition: opacity 0.2s ease;
    opacity: 0;
    visibility: hidden;

    position: fixed;
    background-color: var(--header-bg);
    color: var(--header-link);
    top: 65px;
    left: 0;
    right: 0;
    padding: calc(var(--space)/2) 0;
    z-index: 100;

    border-top: 1px solid currentColor;
  }



@media screen and (max-width: 1080px) {

    .page{
      grid-template-columns: minmax(0,1fr);
      padding-top: calc(var(--space)*2);


      &__sidebar{

        position: fixed;
        top: calc( (2rem + (var(--header-padding))*2 ) *2 );
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 99;

        padding: var(--space);
        background-color: var(--body-bg);
      }
    }

    .sidebarMobileToggle{
      opacity: 1;
      visibility: visible;

      top: calc( 2rem + (var(--header-padding))*2 );

      border-top: 2px solid var(--body-bg);
    }
  }

</style>

<page-query>
query ($id: ID!) {
  doc: docPage (id: $id) {
  	id
    title
    headings (depth: h1) {
      value
    }
    subtitles: headings {
      depth
      value
      anchor
    }
    content
  }
}
</page-query>


<script>
import items from '@/data/doc-links.yaml'

export default {

	components: {
      SidebarDocs: () => import("~/components/SidebarDocs.vue"),
      SidebarContent: () => import("~/components/SidebarContent.vue"),
      Banner: () => import("~/components/Banner.vue"),
      NavIcon: () => import('~/components/NavIcon.vue'),
      IconGithub: () => import('@/assets/images/IconGithub.svg'),
	  },

  data(){
    return {
      items: this.setBranchOpenLabel(this.initOpenLabel(items)),
      // linksList: this.listOfLinks(items),
    }
  },

  methods: {
    initOpenLabel(list) {
      return list.map(item => {
        if (item.items) {
          item.items = this.initOpenLabel(item.items)
        }
        return {...item, isOpen: Object.prototype.hasOwnProperty.call(item, 'link') && this.$route.path === item.link}
      })
    },
    hasOpenChildren(list) {
      return list.find(item =>{
        return item.isOpen
      }) ? true : false
    },
    setBranchOpenLabel(list) {
      return list.map(item => {
        if (item.items && !item.isOpen) {
          item.items = this.setBranchOpenLabel(item.items)
          item.isOpen = this.hasOpenChildren(item.items)
        }
        return {...item}
      })
    },


    // flat(object) {
    //     return Object
    //         .values(object)
    //         .reduce((r, v) => r.concat(v && typeof v === 'object' ? this.flat(v) : v), []);
    // },

    // clean(object) {
    //   for (let value of object.values()) {
    //       if(value.title && !value.link) {
    //         delete value.title
    //       }
    //       if(value.isOpen != 'undefined') {
    //         delete value.isOpen
    //       }
    //       if(value.items) {
    //         this.clean(value.items)
    //       }
    //   }
    // },

    // listOfLinks (object) {
    //   this.clean(object);
    //   return object
    // },


  },

  
  computed: {

    currentPath () {
      return this.$route.matched[0].path
    },
    
    editLink () {
      let path = this.currentPath
      if((path.match(new RegExp("/", "g")) || []).length == 1) path = path + '/README'
      return `https://github.com/airalab/robonomics-wiki/blob/master${path}.md`
    },
    


    /* Это для ссылок на пред и след статью */
    // currentIndex () {
    //   return this.linksList.findIndex(item => {
    //     return item.link.replace(/\/$/, '') === this.$route.path.replace(/\/$/, '')
    //   })
    // },

    // nextPage () {
    //   return this.listOfLinks[this.currentIndex + 1]
    // },

    // previousPage () {
    //   return this.listOfLinks[this.currentIndex - 1]
    // }
    /* Это для ссылок на пред и след статью [конец] */



  },

	metaInfo () {
	    const { title, headings } = this.$page.doc
	    return {
	      title: title || (headings.length ? headings[0].value : undefined)
	    }
	  },

  updated: function(){

    // for (let value of this.items.values()) {
    //     console.log(value.title);
    // }


    //Hide popup mobile menu after clickcing (cause - no real page reload in Gridsome)
    document.querySelectorAll('.menu__link').forEach(function(el) {
      
      el.addEventListener('click', function(event){
        event.target.closest('.page__sidebar').classList.add('hiddenMobile');
        var id = event.target.closest('.page__sidebar').id;
        console.log(id);
        
        document.querySelectorAll('.sectionToggler').forEach(function(el) {
          if(el.dataset.show == id) {
            el.classList.add('open');
            el.classList.remove('close');
          }
        });
      })
    });
  }
}


</script>
