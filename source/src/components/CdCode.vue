<template>
	<div class="window" v-draggable="draggable" v-if="show">
		<div class="window-header">
			<button id="copy-code" v-clipboard="code"><i class="material-icons" style="font-size: 40px">content_copy</i></button>
			<div class="title">{{ title }}</div>
			<button class="close" v-if="closeable" @click="show = false"><i class="material-icons">close</i></button>
			<div class="filler" v-else></div>
			<div class="language">{{ language }}</div>
		</div>
    <div class="window-body">
      <textarea class="code-input" spellcheck="false" :readonly="!writeable" v-model="code" onkeydown="if(event.keyCode===9){var v=this.value,s=this.selectionStart,e=this.selectionEnd;this.value=v.substring(0, s)+'\t'+v.substring(e);this.selectionStart=this.selectionEnd=s+1;return false;}"></textarea>
  		<pre class="code-output" :class="language">
  			<code v-html="highlightedCode(language)"></code>
  		</pre>
      <pre class="invis">
        <code class="invis" v-html="highlightedCode(language)+'\n'"></code>
      </pre>
    </div>
	</div>
</template>

<script>
import Vue from 'vue'
import VueClipboards from 'vue-clipboards'

Vue.use(VueClipboards)

Vue.directive('draggable', {
  bind: function (el, binding) {
		if (binding.value) {
			el.style.position = 'absolute';
	    var startX, startY, initialMouseX, initialMouseY;

	    function mousemove(e) {
	      var dx = e.clientX - initialMouseX;
	      var dy = e.clientY - initialMouseY;
	      el.style.top = startY + dy + 'px';
	      el.style.left = startX + dx + 'px';
	      return false;
	    }

	    function mouseup() {
	      document.removeEventListener('mousemove', mousemove);
	      document.removeEventListener('mouseup', mouseup);
	    }

			let elements = []
			setTimeout(()=>{
				let elements2 = document.getElementsByClassName('window')
				for (let i in elements2) {
					if (elements2[i].style != undefined) {
						elements2[i].style.zIndex = i*1+1
						elements.push(elements2[i])
					}
				}
			}, 100)

	    el.children[0].addEventListener('mousedown', function(e) {
				let topZindex = 1
				if (elements[0].style != undefined) {
					let sortedElements = elements.sort((a, b) => {
						return a.style.zIndex > b.style.zIndex ? 1 : -1
					})
					topZindex = sortedElements[0].style.zIndex
				}
				el.style.zIndex = topZindex*1 + 1
	      startX = el.offsetLeft;
	      startY = el.offsetTop;
	      initialMouseX = e.clientX;
	      initialMouseY = e.clientY;
	      document.addEventListener('mousemove', mousemove);
	      document.addEventListener('mouseup', mouseup);
	      return false;
	    });
		}
  }
})

export default {
  name: 'CdCode',
  props: {
    value: {
      type: String,
      default: ''
    },
    language: {
      type: String,
      default: 'html'
    },
    title: {
      type: String,
      required: true
    },
    writeable: {
      type: Boolean,
      default: false
    },
		draggable: {
			type: Boolean,
			default: false
		},
		closeable: {
			type: Boolean,
			default: false
		}
  },
  data () {
    let tmp2 = ''
    if (this.value != '') {
      let tmp = this.value.split('\n')
      tmp.splice(0, 1)
      tmp.splice(tmp.length-1)
      tmp2 = tmp.join('\n')
    }
    return {
      code: tmp2,
			show: true
    }
  },
  computed: {
    highlightedCode1 () {
      let code = this.code
      let html = Prism.highlight(code, Prism.languages.html)
      return html
    }
  },
  methods: {
    highlightedCode (lang) {
      let code = this.code
			code = code.replace(/(<|>)/g, (el, tag)=> {
				if (tag == '<') {
					return '&lt'
				}
				if (tag == '>') {
					return '&gt'
				}
			})
      if (lang == 'html') {
        let html = code
        html = html.replace(/&lt(.+?)&gt/g, this.styleHTML)
        html = html.replace(/<span class="tag-punct">&lt<\/span><span class="tag">(!--[\s\S]*?--)<\/span><span class="tag-punct">&gt<\/span>/g, (el, code) => {
          return '<span class="comment">&lt'+code.replace(/<span class=".+?">|<\/span>/g, '')+'&gt</span>'
        })
        return html
      }
      else if (lang == 'javascript') {
        let html = code
        html = html.replace(/.+/g, this.styleJS)
        return html
      }
      else {
        return code
      }
    },
    styleHTML (el, tag) {
      let tagPunct1 = '<span class="tag-punct">&lt</span>'
      let tagPunct2 = '<span class="tag-punct">&gt</span>'
      let tagName = '<span class="tag">'+tag.split(' ')[0]+'</span>'
      let attributes = ''
      if (tag == '!DOCTYPE html') {
        tagName = '<span class="doctype">'+tag+'</span>'
      }
      if (tag.split(' ')[1] && tag != '!DOCTYPE html') {
        let c = tag.split(/ /)
        let c2 = ''
        for (let i = 1; i < c.length; i++) {
          c2 = c2 + ' ' + c[i]
        }
        c2 = c2.replace(/(")(.+?)(")/g, '<span class="punct">$1</span><span class="string">$2</span><span class="punct">$3</span>')
        c2 = c2.replace(/(.+?)(=)(<span)/g, '$1<span class="punct">$2</span>$3')
        attributes = '<span class="attribute">'+c2+'</span>'
      }
      return tagPunct1+tagName+attributes+tagPunct2
    },
    styleJS (el) {
      let styled = el
      if (/(\/\/.+)/g.test(el)) {
        styled = el.replace(/\/\/.+/g, '<span class="comment">$&</span>')
      } else {
				styled = styled.replace(/([0-9])/g, '<span class="number">$1</span>')
	      styled = styled.replace(/(var|let|const|Boolean|String|Number|Object|Array)/g, '<span class="type">$1</span>')
	      styled = styled.replace(/(import|new |return|function|export default|export|from| in |for |if |else )/g, '<span class="function">$1</span>')
	      styled = styled.replace(/(true)/g, '<span class="true">$1</span>')
	      styled = styled.replace(/(false)/g, '<span class="false">$1</span>')
	      styled = styled.replace(/(.+?) \((.*)\)(?= {)/g, '<span class="function">$1</span> (<span class="args">$2</span>)')
	      styled = styled.replace(/([;:,.!^*_+\-=|?(){}\[\]])(?!['"])/g, '<span class="punct">$1</span>')
	      styled = styled.replace(/ (?!(class=))(['`])(.*?)['`]/g, '<span class="string"> <span class="punct">$2</span>$3<span class="punct">$2</span></span>')
	      styled = styled.replace(/(.+)/g, '<span class="var">$1</span>')
			}
      return styled
    }
  }
}
</script>

<style lang="css">
  .window {
    width: 100%;
    height: auto;
    border-radius: 3px;
    position: relative;
    background: #F0F0F0;
    overflow: auto;
    box-shadow: 0 5px 10px 0px rgba(0, 0, 0, 0.4);
		z-index: 5;
  }

  .window-header {
    height: 40px;
    background: #DCDCDC;
  }
	.window-header #copy-code {
		border-radius: 50%;
		margin: 0;
		padding: 0;
		border: none;
		box-shadow: none;
		background: transparent;
		z-index: 4;
		position: absolute;
		right: 0;
		top: 40px;
		padding: 10px;
		opacity: .3;
		transition: background .3s ease;
	}
	.window-header #copy-code:hover {
		background: rgba(0, 0, 0, .1)
	}
  .window-header .title {
    float: left;
    font-size: 16px !important;
    font-weight: 600;
    margin: 0;
    position: relative;
    top: 12px;
    left: 12px;
    opacity: .7;
  }
  .window-header .language {
    float: right;
    font-size: 16px;
    font-weight: 600;
    margin: 0;
    position: relative;
    top: 12px;
    right: 5px;
    opacity: .7;
		text-transform: uppercase;
  }
	.window-header .close {
		width: 36px;
		height: 36px;
		margin: 0;
		margin-top: 2px;
		margin-right: 2px;
		padding: 0;
		border: none;
		float: right;
		background: transparent;
		cursor: pointer;
		border-radius: 20px;
		transition: background .3s ease;
	}
	.window-header .close:hover {
		background: rgba(0, 0, 0, .1);
	}
	.window-header .close i {
		float: right;
		font-size: 20px;
		font-weight: 600;
		margin: 0;
		position: relative;
		top: 1px;
		right: 8px;
		opacity: .7;
	}
	.window-header .filler {
		width: 7px;
		height: 40px;
		float: right;
	}

  .window-body {
    position: relative;
  }

  .window-body textarea, .window-body code {
    width: 100%;
    height: auto;
    min-height: 100%;
    font-size: 20px;
    font-weight: 600;
    border: 0;
    margin: 0;
    top: 0;
    left: 0;
    padding: 20px !important;
    line-height: 27px;
    position: absolute;
    font-family: Roboto,Consolas,Liberation Mono,Courier,monospace;
    overflow: visible;
    -webkit-transition: all 0.5s ease-in-out;
    transition: all 0.5s ease-in-out;
    opacity: 1;
    tab-size: 2;
  }
  .window-body code.invis {
    position: static;
    opacity: 0;
  }
  .window-body pre.invis {
    min-height: 90px;
  }

  .window-body textarea {
    background: transparent !important;
    z-index: 2;
    height: auto;
    resize: none;
    color: #666666;
    text-shadow: 0px 0px 0px transparent;
    text-fill-color: transparent;
    -webkit-text-fill-color: transparent;
    white-space: pre-wrap;
    white-space: -moz-pre-wrap;
    white-space: -pre-wrap;
    white-space: -o-pre-wrap;
    word-wrap: break-word;
    overflow: hidden;
  }
  .window-body textarea::-webkit-input-placeholder {
    color: black;
  }
  .window-body textarea:focus {
    outline: 0;
    border: 0;
    box-shadow: none;
  }
  .window-body code {
    z-index: 1;
  }

  .window-body pre.code-output {
    white-space: pre-wrap;
    white-space: -moz-pre-wrap;
    white-space: -pre-wrap;
    white-space: -o-pre-wrap;
    word-wrap: break-word;
    height: 0;
    margin: 0;
  }
  .window-body pre code {
    background: #F0F0F0;
    color: #666666;
  }

  .link {
    text-decoration: underline;
  }

  .html .comment {
    color: #19B357;
		font-style: italic;
  }
  .html .tag,
  .html .tag-punct {
    color: #ab47bc;
  }
  .html .doctype {
    color: #666666;
  }
  .html .attribute {
    color: #2196f3;
  }
  .html .punct {
    color: #F9A825;
  }
  .html .string {
    color: #e57373;
  }

  .javascript .comment {
    color: #19B357;
		font-style: italic;
  }
  .javascript .number {
    color: #19B357;
  }
  .javascript .function {
    color: #ab47bc;
  }
  .javascript .args {
    color: #e57373;
  }
  .javascript .string {
    color: #e57373;
  }
  .javascript .punct {
    color: #5D65D5;
  }
  .javascript .type {
    color: #F9A825;
  }
  .javascript .true {
    color: #19B357;
  }
  .javascript .false {
    color: #EC464F;
  }
  .javascript .var {
    color: #2196f3;
  }

</style>
