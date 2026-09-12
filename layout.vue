<template>
  <section class="monaco-editor-shell" :class="{ 'theseed-dark-mode': theme === 'vs-dark', 'is-expanded': isExpanded }">
    <div class="format-toolbar" role="toolbar" :aria-label="editorText('toolbar')">
      <div class="toolbar-group">
        <ContextMenu ref="colorMenu" :title="editorText('fontSize')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('fontSize')"><font-awesome-icon :icon="icons.fontSize"/><font-awesome-icon class="toolbar-caret" :icon="icons.caretDown"/></GeneralButton>
            <template #menu><div><GeneralButton v-for="item in fontSizes" :key="item.value" type="event" class="font-size-option" @click="applyFormat('size', item.value)"><span>{{editorText(item.key)}}</span><span class="font-size-preview" :class="`font-size-preview--${item.value.replace('-', 'minus')}`">Aa</span></GeneralButton></div></template>
        </ContextMenu>
        <ContextMenu :title="editorText('fontColor')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('fontColor')"><font-awesome-icon :icon="icons.palette"/></GeneralButton>
          <template #menu>
            <div class="color-menu">
              <label for="editor-color">{{editorText('fontColor')}}</label>
              <div class="color-field"><InputField id="editor-color" v-model="color" type="color" class="color-picker-input"/><InputField v-model="color" type="text" class="color-text-input"/></div>
              <br><CheckBox v-model="darkColorEnabled">{{editorText('darkColor')}}</CheckBox>
              <div :key="darkColorEnabled" class="color-field"><InputField id="editor-dark-color" v-model="darkColor" type="color" class="color-picker-input" :disabled="!darkColorEnabled"/><InputField v-model="darkColor" type="text" class="color-text-input" :disabled="!darkColorEnabled"/></div>
              <div class="insert-actions"><GeneralButton submit type="event" @click="insertColor">{{editorText('insert')}}</GeneralButton></div>
            </div>
          </template>
        </ContextMenu>
        <div class="toolbar-divider"/>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('bold')" @click="applyFormat('bold')"><font-awesome-icon :icon="icons.bold"/></GeneralButton>
        <GeneralButton class="toolbar-button italic" type="event" :title="editorText('italic')" @click="applyFormat('italic')"><font-awesome-icon :icon="icons.italic"/></GeneralButton>
        <GeneralButton class="toolbar-button strike" type="event" :title="editorText('strike')" @click="applyFormat('strike')"><font-awesome-icon :icon="icons.strike"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('subscript')" @click="applyFormat('subscript')"><font-awesome-icon :icon="icons.subscript"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('superscript')" @click="applyFormat('superscript')"><font-awesome-icon :icon="icons.superscript"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('underline')" @click="applyFormat('underline')"><font-awesome-icon :icon="icons.underline"/></GeneralButton>
      </div>
      <div class="toolbar-divider"/>
      <div class="toolbar-group">
        <ContextMenu :title="editorText('heading')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', 'heading-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('heading')"><font-awesome-icon :icon="icons.heading"/><font-awesome-icon class="toolbar-caret" :icon="icons.caretDown"/></GeneralButton>
          <template #menu><div><GeneralButton v-for="level in 6" :key="level" type="event" class="heading-option" @click="applyHeading(level)"><span>{{editorText('headingLevel', { level })}}</span><span class="heading-preview">{{'='.repeat(level)}} {{editorText('headingLevel', { level })}} {{'='.repeat(level)}}</span></GeneralButton></div></template>
        </ContextMenu>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('list')" @click.stop="applyLinePrefix('* ')"><font-awesome-icon :icon="icons.list"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('orderedList')" @click.stop="applyLinePrefix('1. ')"><font-awesome-icon :icon="icons.listNumbers"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('quote')" @click="applyLinePrefix('> ')"><font-awesome-icon :icon="icons.quote"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('footnote')" @click.stop="applyFormat('footnote')"><font-awesome-icon :icon="icons.note"/></GeneralButton>
        <GeneralButton class="toolbar-button" type="event" :title="editorText('comment')" @click.stop="applyLinePrefix('## ')"><font-awesome-icon :icon="icons.hashtag"/></GeneralButton>
        <ContextMenu :title="editorText('macro')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('macro')"><font-awesome-icon :icon="icons.macro"/></GeneralButton>
          <template #menu><div class="macro-menu"><label for="editor-macro">{{editorText('content')}}</label><SelectMenu id="editor-macro" v-model="macro.type"><option v-for="item in macroTypes" :key="item.value" :value="item.value">{{editorText(item.key)}}</option></SelectMenu><br><InputField v-model="macro.value" type="text"/><div class="insert-actions"><GeneralButton submit type="event" @click="insertMacro">{{editorText('insert')}}</GeneralButton></div></div></template>
        </ContextMenu>
        <ContextMenu :title="editorText('syntaxHighlight')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('syntaxHighlight')"><font-awesome-icon :icon="icons.code"/></GeneralButton>
          <template #menu><div><label for="editor-language">{{editorText('language')}}</label><SelectMenu id="editor-language" v-model="language"><option value="">{{editorText('syntaxHighlight')}}</option><option v-for="item in languages" :key="item.value" :value="item.value">{{item.label}}</option></SelectMenu><div class="insert-actions"><GeneralButton submit type="event" @click="insertCodeBlock">{{editorText('insert')}}</GeneralButton></div></div></template>
        </ContextMenu>
      </div>
      <div class="toolbar-divider"/>
      <div class="toolbar-group">
        <ContextMenu :title="editorText('link')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('link')"><font-awesome-icon :icon="icons.link"/></GeneralButton>
          <template #menu>
            <div class="insert-menu link-menu">
              <CheckBox class="menu-check" v-model="link.targetEnabled">{{editorText('destination')}}</CheckBox>
              <InputField v-model="link.target" type="text"/>
              <CheckBox class="menu-check" v-model="link.contentEnabled">{{editorText('content')}}</CheckBox>
              <InputField v-model="link.content" type="text"/>
              <div class="insert-actions"><GeneralButton submit type="event" @click="insertLink">{{editorText('insert')}}</GeneralButton></div>
            </div>
          </template>
        </ContextMenu>
        <ContextMenu :title="editorText('image')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('image')"><font-awesome-icon :icon="icons.image"/></GeneralButton>
          <template #menu>
            <div class="insert-menu image-menu">
              <label>{{editorText('fileName')}}</label>
              <InputField ref="imageName" v-model="image.name" type="text" required :hasError="imageNameError" list="image-file-suggestions" @update:modelValue="imageNameError = false"/>
              <datalist id="image-file-suggestions"><option v-for="extension in imageExtensions" :key="extension" :value="extension"/></datalist>
              <div class="field-grid">
                <CheckBox class="menu-check" v-model="image.widthEnabled">{{editorText('width')}}</CheckBox>
                <CheckBox class="menu-check" v-model="image.heightEnabled">{{editorText('height')}}</CheckBox>
                <InputField v-model="image.width" type="text" :disabled="!image.widthEnabled"/>
                <div class="unit-field"><InputField v-model="image.height" type="text" :disabled="!image.heightEnabled"/><SelectMenu v-model="image.heightUnit" :disabled="!image.heightEnabled"><option value="px">px</option><option value="%">%</option></SelectMenu></div>
              </div>
              <div class="field-grid">
                <CheckBox class="menu-check" v-model="image.alignEnabled">{{editorText('align')}}</CheckBox>
                <CheckBox class="menu-check" v-model="image.fitEnabled">{{editorText('fit')}}</CheckBox>
                <SelectMenu v-model="image.align" :disabled="!image.alignEnabled"><option value="">{{editorText('default')}}</option><option value="left">{{editorText('left')}}</option><option value="center">{{editorText('center')}}</option><option value="right">{{editorText('right')}}</option></SelectMenu>
                <SelectMenu v-model="image.fit" :disabled="!image.fitEnabled"><option value="">{{editorText('default')}}</option><option value="contain">contain</option><option value="cover">cover</option><option value="fill">fill</option><option value="none">none</option><option value="scale-down">scale-down</option></SelectMenu>
              </div>
              <div class="field-grid">
                <CheckBox class="menu-check" v-model="image.backgroundEnabled">{{editorText('backgroundColor')}}</CheckBox>
                <CheckBox class="menu-check" v-model="image.radiusEnabled">{{editorText('radius')}}</CheckBox>
                <div class="color-field image-background-field"><InputField v-model="image.background" type="color" class="color-picker-input" :disabled="!image.backgroundEnabled"/><InputField v-model="image.background" type="text" class="color-text-input" :disabled="!image.backgroundEnabled"/></div>
                <InputField v-model="image.radius" type="text" :disabled="!image.radiusEnabled"/>
              </div>
              <div class="field-grid">
                <CheckBox class="menu-check" v-model="image.themeEnabled">{{editorText('themeOnly')}}</CheckBox>
                <span class="menu-check">{{editorText('disableAntialiasing')}}</span>
                <SelectMenu v-model="image.theme" :disabled="!image.themeEnabled"><option value="">{{editorText('themeSelect')}}</option><option value="light">{{editorText('lightTheme')}}</option><option value="dark">{{editorText('darkTheme')}}</option></SelectMenu>
                <CheckBox class="menu-check" v-model="image.antiAliasingEnabled">{{editorText('use')}}</CheckBox>
              </div>
              <div class="insert-actions"><GeneralButton submit type="event" @click="insertImage">{{editorText('insert')}}</GeneralButton></div>
            </div>
          </template>
        </ContextMenu>
        <ContextMenu :title="editorText('video')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
          <GeneralButton class="toolbar-button" type="event" :title="editorText('video')"><font-awesome-icon :icon="icons.video"/></GeneralButton>
          <template #menu>
            <div class="insert-menu video-menu">
              <label>{{editorText('destination')}}</label>
              <InputField v-model="video.url" type="text"/>
              <div class="field-grid video-size-grid">
                <CheckBox class="menu-check" v-model="video.widthEnabled">{{editorText('width')}}</CheckBox>
                <CheckBox class="menu-check" v-model="video.heightEnabled">{{editorText('height')}}</CheckBox>
                <div class="unit-field"><InputField v-model="video.width" type="text" :disabled="!video.widthEnabled"/><SelectMenu v-model="video.widthUnit" :disabled="!video.widthEnabled"><option value="px">px</option><option value="%">%</option></SelectMenu></div>
                <div class="unit-field"><InputField v-model="video.height" type="text" :disabled="!video.heightEnabled"/><SelectMenu v-model="video.heightUnit" :disabled="!video.heightEnabled"><option value="px">px</option><option value="%">%</option></SelectMenu></div>
              </div>
              <div class="external-sites">
                <p>{{editorText('externalSites')}}</p>
                <ul class="external-sites-list"><li>{{editorText('naverTv')}}</li><li>{{editorText('niconico')}}</li><li>{{editorText('vimeo')}}</li><li>{{editorText('youtube')}}</li></ul>
              </div>
              <div class="insert-actions"><GeneralButton submit type="event" @click="insertVideo">{{editorText('insert')}}</GeneralButton></div>
            </div>
          </template>
        </ContextMenu>
        <div class="toolbar-divider"/>
      </div>
      <div class="toolbar-spacer"/>
      <ContextMenu :title="editorText('tools')" :popperClass="['context-menu', 'max-size', 'editor-context-menu', { 'theseed-dark-mode-context-menu': theme === 'vs-dark' }]">
        <GeneralButton class="toolbar-button" type="event" :title="editorText('tools')"><font-awesome-icon :icon="icons.tools"/></GeneralButton>
        <template #menu><div class="document-tools-menu">
          <GeneralButton type="event" @click="saveDraft"><font-awesome-icon :icon="icons.save"/><span>{{editorText('saveDraft')}}</span></GeneralButton>
          <GeneralButton type="event" @click="loadDraft"><font-awesome-icon :icon="icons.load"/><span>{{editorText('loadDraft')}}</span></GeneralButton>
          <GeneralButton type="event" @click="copyDocumentTitle"><font-awesome-icon :icon="icons.copy"/><span>{{editorText('copyTitle')}}</span></GeneralButton>
          <GeneralButton type="event" @click="copyRaw"><font-awesome-icon :icon="icons.copy"/><span>{{editorText('copyRaw')}}</span></GeneralButton>
        </div></template>
      </ContextMenu>
      <GeneralButton class="toolbar-button" type="event" :title="editorText(isExpanded ? 'collapse' : 'expand')" :aria-pressed="isExpanded" @click="toggleExpanded"><font-awesome-icon :icon="isExpanded ? icons.contract : icons.expand"/></GeneralButton>
      <div class="toolbar-divider"/>
      <GeneralButton class="toolbar-button toolbar-label" type="event" :title="editorText('compare')" :aria-pressed="compareMode" @click="toggleCompare"><font-awesome-icon :icon="icons.compare"/><span>{{editorText('compare')}}</span></GeneralButton>
    </div>

    <div v-show="!compareMode" ref="div" class="editor-surface"/>
    <div v-show="compareMode" ref="compareDiv" class="editor-surface compare-surface"/>
    <footer class="editor-statusbar">
      <div class="toolbar-divider"/>
      <span>{{editorText('lineColumn', cursorPosition)}}</span>
      <div class="toolbar-divider"/>
      <span>{{editorText('characterCount', { count: characterCount })}} <span class="muted">({{characterCountNoSpaces}})</span></span>
      <div class="toolbar-divider"/>
      <span>{{editorText('lineCount', { count: lineCount })}}</span>
    </footer>

  </section>
</template>
<script>
import { h, markRaw, nextTick } from 'vue'
import { hideAllPoppers } from 'floating-vue'
import GeneralButton from '@/components/GeneralButton'
import ContextMenu from '@/components/contextMenu'
import InputField from '@/components/form/inputField'
import SelectMenu from '@/components/selectMenu'
import CheckBox from '@/components/form/checkBox'
import { toast } from 'vue-sonner'
import {
  faBold,
  faCaretDown,
  faCode,
  faCopy,
  faFileArrowDown,
  faFloppyDisk,
  faHashtag,
  faHeading,
  faCompress,
  faExpand,
  faImage,
  faItalic,
  faLink,
  faList,
  faListOl,
  faNoteSticky,
  faPalette,
  faPuzzlePiece,
  faQuoteLeft,
  faStrikethrough,
  faSubscript,
  faSuperscript,
  faTableColumns,
  faTextHeight,
  faToolbox,
  faUnderline,
  faVideo,
  faXmark
} from '@fortawesome/free-solid-svg-icons'
import { default as namumarkRegister } from './namu/vs/languages/namumark'
import { QuickAccess } from './namu/toolbar/quickaccess'
import koLocale from './locale/ko.json'
import enLocale from './locale/en.json'
import editorWorker from 'monaco-editor/esm/vs/editor/editor.worker?worker'

export default {
  components: {
    FontAwesomeIcon: {
      props: { icon: Object },
      render() {
        const [width, height, , , path] = this.icon.icon
        const paths = Array.isArray(path) ? path : [path]
        return h('svg', {
          viewBox: `0 0 ${width} ${height}`,
          'aria-hidden': 'true',
          focusable: 'false',
          fill: 'currentColor'
        }, paths.map(value => h('path', { d: value })))
      }
    },
    ContextMenu,
    InputField,
    SelectMenu,
    CheckBox,
    GeneralButton
  },
  pluginInfo: {
    name: 'thetree-monaco',
    label: '편집기',
    buttons: []
  },
  data() {
    return {
      icons: {
        bold: faBold,
        caretDown: faCaretDown,
        code: faCode,
        copy: faCopy,
        load: faFileArrowDown,
        save: faFloppyDisk,
        hashtag: faHashtag,
        heading: faHeading,
        compare: faTableColumns,
        contract: faCompress,
        expand: faExpand,
        fontSize: faTextHeight,
        image: faImage,
        italic: faItalic,
        link: faLink,
        list: faList,
        listNumbers: faListOl,
        macro: faPuzzlePiece,
        note: faNoteSticky,
        palette: faPalette,
        quote: faQuoteLeft,
        strike: faStrikethrough,
        subscript: faSubscript,
        superscript: faSuperscript,
        tools: faToolbox,
        underline: faUnderline,
        video: faVideo,
        close: faXmark
      },
      editor: null,
      monaco: null,
      quickaccess: null,
      originalContent: '',
      isExpanded: false,
      compareMode: false,
      diffEditor: null,
      fontSize: '1',
        fontSizes: [
          { value: '-2', key: 'smaller2' },
          { value: '-1', key: 'smaller1' },
          { value: '1', key: 'larger1' },
          { value: '2', key: 'larger2' }
        ],
      color: '#000000',
      darkColor: '#ffffff',
      darkColorEnabled: false,
      link: {
        target: '',
        content: '',
        targetEnabled: false,
        contentEnabled: false
      },
      image: {
        name: '',
        width: '',
        height: '',
        heightUnit: 'px',
        align: '',
        fit: '',
        background: '#ffffff',
        radius: '',
        theme: '',
        widthEnabled: false,
        heightEnabled: false,
        alignEnabled: false,
        fitEnabled: false,
        backgroundEnabled: false,
        radiusEnabled: false,
        themeEnabled: false,
        antiAliasing: false,
        antiAliasingEnabled: false
      },
      imageExtensions: ['.jpg', '.jpeg', '.png', '.gif', '.webp'],
      imageNameError: false,
      video: {
        url: '',
        width: '',
        height: '',
        widthUnit: 'px',
        heightUnit: 'px',
        widthEnabled: false,
        heightEnabled: false
      },
      language: '',
      macro: {
        type: 'include',
        value: ''
      },
      macroTypes: [
        { value: 'include', key: 'include' },
        { value: 'age', key: 'age' },
        { value: 'date', key: 'date' },
        { value: 'dday', key: 'dday' },
        { value: '목차', key: 'tableOfContents' },
        { value: '각주', key: 'footnoteMacro' },
        { value: 'br', key: 'br' },
        { value: 'clearfix', key: 'clearfix' }
      ],
      languages: [
        { value: 'basic', label: 'BASIC' },
        { value: 'cpp', label: 'C++' },
        { value: 'csharp', label: 'C#' },
        { value: 'css', label: 'CSS' },
        { value: 'diff', label: 'Diff' },
        { value: 'erlang', label: 'Erlang' },
        { value: 'go', label: 'Go' },
        { value: 'html', label: 'HTML' },
        { value: 'java', label: 'Java' },
        { value: 'javascript', label: 'JavaScript' },
        { value: 'json', label: 'JSON' },
        { value: 'kotlin', label: 'Kotlin' },
        { value: 'lisp', label: 'Lisp' },
        { value: 'lua', label: 'Lua' },
        { value: 'markdown', label: 'Markdown' },
        { value: 'objective-c', label: 'Objective-C' },
        { value: 'patch', label: 'Patch' },
        { value: 'perl', label: 'Perl' },
        { value: 'php', label: 'PHP' },
        { value: 'python', label: 'Python' },
        { value: 'ruby', label: 'Ruby' },
        { value: 'sql', label: 'SQL' },
        { value: 'typescript', label: 'TypeScript' },
        { value: 'xml', label: 'XML' }
      ],
      characterCount: 0,
      characterCountNoSpaces: 0,
      lineCount: 1,
      cursorPosition: {
        line: 1,
        column: 1
      }
    }
  },
  computed: {
    theme() {
      return this.$store.state.currentTheme === 'dark' ? 'vs-dark' : 'vs'
    }
  },
  watch: {
    theme(newValue) {
      this.monaco.editor.setTheme(newValue)
    }
  },
  async mounted() {
    window.addEventListener('keydown', this.handleKeydown)
    const monaco = await import('monaco-editor')

    namumarkRegister(monaco)

    self.MonacoEnvironment = {
      getWorker(_, label) {
        return new editorWorker()
      }
    }

    this.monaco = markRaw(monaco)
    this.originalContent = this.$store.state.viewData.content
    this.editor = markRaw(monaco.editor.create(this.$refs.div, {
      language: 'namumark',
      automaticLayout: true,
      wordWrap: true,
      renderWhitespace: 'all',
      fontFamily: 'D2Coding, Consolas, "나눔고딕코딩", "Courier New", monospace',
      value: this.$store.state.viewData.content,
      minimap: {
        enabled: false
      },
      theme: this.theme
    }))

    this.quickaccess = new QuickAccess(this.editor, this.monaco)
    this.editor.onDidChangeModelContent(() => this.updateStatus())
    this.editor.onDidChangeCursorPosition(() => this.updateStatus())
    this.updateStatus()
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown)
  },
  methods: {
    currentLanguage() {
      if(this.$i18next && this.$i18next.language)
        return String(this.$i18next.language).split('-')[0]
      if(typeof document !== 'undefined') {
        const match = document.cookie.match(/(?:^|; )thetree\.lang=([^;]*)/)
        if(match && match[1]) return decodeURIComponent(match[1]).split('-')[0]
      }
      const rawLang = this.$store.state.config?.lang || 'ko'
      return String(rawLang).split('-')[0]
    },
    currentLocale() {
      return this.currentLanguage() === 'en' ? 'en-US' : 'ko-KR'
    },
    editorText(key, values = {}) {
      const locale = this.currentLanguage() === 'en' ? enLocale : koLocale
      let text = locale[key] || koLocale[key] || key
      return text.replace(/\{(\w+)\}/g, (_, name) => values[name] ?? `{${name}}`)
    },
    handleKeydown(event) {
      if(event.ctrlKey && event.key.toLowerCase() === 's') {
        event.preventDefault()
        this.saveDraft()
      } else if(event.ctrlKey && event.key.toLowerCase() === 'o') {
        event.preventDefault()
        this.loadDraft()
      } else if(event.key === 'F11') {
        event.preventDefault()
        this.toggleExpanded()
      } else if(event.key === 'Escape' && this.isExpanded) {
        this.isExpanded = false
        this.editor?.layout()
        this.diffEditor?.layout()
      }
    },
    getValue() {
      return this.editor.getValue()
    },
    setValue(value) {
      this.editor.setValue(value)
      this.updateStatus()
    },
    onButtonClick(name) {
      this.applyFormat(name)
    },
    applyFormat(name) {
      const value = arguments[1]
      const descriptor = {
        bold: { bracket: `'''` },
        italic: { bracket: `''` },
        strike: { bracket: `~~` },
        underline: { bracket: `__` },
        subscript: { bracket: `,,` },
        superscript: { bracket: `^^` },
        code: { bracket: `{{{` },
        link: {
          bracket: {
            open: `[[`,
            close: `]]`
          }
        },
        file: {
          bracket: {
            open: `[[파일:`,
            close: `]]`
          }
        },
        footnote: {
          bracket: {
            open: `[* `,
            close: `]`
          }
        },
        include: {
          bracket: {
            open: `[include(`,
            close: `)]`
          }
        },
        video: {
          bracket: {
            open: `[youtube(`,
            close: `)]`
          }
        }
      }[name] || { bracket: { open: '', close: '' } }
      if(name === 'size') descriptor.bracket = { open: `{{{+${value} `, close: '}}}' }
      if(name === 'color') descriptor.bracket = { open: `{{{#${value.color.replace('#', '')}${value.darkColor ? `,#${value.darkColor.replace('#', '')}` : ''} `, close: '}}}' }
      this.quickaccess.apply(descriptor)
      this.updateStatus()
    },
    insertColor() {
      this.applyFormat('color', { color: this.color, darkColor: this.darkColorEnabled ? this.darkColor : null })
      hideAllPoppers()
    },
    insertCodeBlock() {
      const selection = this.editor.getSelection()
      const model = this.editor.getModel()
      const selected = model.getValueInRange(selection)
      const text = `{{{#!syntax ${this.language}\n${selected}\n}}}`
      this.editor.executeEdits('toolbar', [{ range: selection, text }])
      this.editor.setPosition({ lineNumber: selection.startLineNumber + 1, column: 1 })
      this.editor.focus()
      hideAllPoppers()
    },
    insertMacro() {
      const value = this.macro.value ? `(${this.macro.value})` : '()'
      const selection = this.editor.getSelection()
      const selected = this.editor.getModel().getValueInRange(selection)
      const text = `[${this.macro.type}${value}${selected ? ` ${selected}` : ''}]`
      this.editor.executeEdits('toolbar', [{ range: selection, text }])
      this.editor.focus()
      hideAllPoppers()
    },
    insertLink() {
      const selection = this.editor.getSelection()
      const selected = this.editor.getModel().getValueInRange(selection)
      const target = this.link.targetEnabled ? this.link.target : this.link.target || selected
      const content = this.link.contentEnabled ? this.link.content : this.link.content || selected
      const text = `[[${target}${content ? `|${content}` : ''}]]`
      this.editor.executeEdits('toolbar', [{ range: selection, text }])
      this.editor.focus()
      this.updateStatus()
      hideAllPoppers()
    },
    insertImage() {
      if(!this.image.name.trim()) {
        this.imageNameError = true
        this.$refs.imageName?.focus()
        return
      }
      const attributes = [
        this.image.widthEnabled && this.image.width && `width=${this.image.width}`,
        this.image.heightEnabled && this.image.height && `height=${this.image.height}${this.image.heightUnit === '%' ? '%' : ''}`,
        this.image.alignEnabled && this.image.align && `align=${this.image.align}`,
        this.image.fitEnabled && this.image.fit && `object-fit=${this.image.fit}`,
        this.image.backgroundEnabled && this.image.background && `bg=${this.image.background.replace('#', '')}`,
        this.image.radiusEnabled && this.image.radius && `radius=${this.image.radius}`,
        this.image.themeEnabled && this.image.theme && `theme=${this.image.theme}`,
        this.image.antiAliasingEnabled && 'rendering=pixelated'
      ].filter(Boolean).join(',')
      const bracket = {
        open: `[[파일:${this.image.name}${attributes ? `|${attributes}` : ''}`,
        close: ']]'
      }
      this.quickaccess.apply({ bracket })
      this.updateStatus()
      hideAllPoppers()
    },
    insertVideo() {
      const service = this.video.url.includes('youtube') ? 'youtube' : this.video.url.includes('nicovideo') ? 'nicovideo' : this.video.url.includes('vimeo') ? 'vimeo' : 'youtube'
      const selection = this.editor.getSelection()
      const attributes = [
        this.video.widthEnabled && this.video.width && `width=${this.video.width}${this.video.widthUnit === '%' ? '%' : ''}`,
        this.video.heightEnabled && this.video.height && `height=${this.video.height}${this.video.heightUnit === '%' ? '%' : ''}`
      ].filter(Boolean).join(',')
      const text = `[${service}(${this.video.url}${attributes ? `,${attributes}` : ''})]`
      this.editor.executeEdits('toolbar', [{ range: selection, text }])
      this.editor.focus()
      this.updateStatus()
      hideAllPoppers()
    },
    getDraftKey() {
      return `thetree-editor-draft:${window.location.pathname}`
    },
    getDocumentInfo() {
      const pageData = this.$store.state.page?.data || {}
      const docObj = pageData.document
      const title = typeof docObj === 'string'
        ? docObj
        : docObj?.namespace
          ? `${docObj.namespace}:${docObj.title}`
          : docObj?.title || this.$store.state.page?.title || this.$route?.params?.page || ''
      const rev = this.$route?.query?.rev || pageData.rev || pageData.revs?.[0]?.rev || ''
      return { title, rev }
    },
    async getActualDocumentTitle(title, rev) {
      if(!title || !rev) return title
      const params = new URLSearchParams({ document: title, rev: String(rev) })
      try {
        const response = await fetch(`/plugins/doc/title?${params}`)
        if(!response.ok) return title
        const data = await response.json()
        return data.title || title
      } catch {
        return title
      }
    },
    async copyText(value) {
      try {
        if(navigator.clipboard?.writeText) {
          await navigator.clipboard.writeText(value)
          return
        }
      } catch {
        // Use the legacy fallback when clipboard permissions reject the modern API.
      }
      const textarea = document.createElement('textarea')
      textarea.value = value
      document.body.appendChild(textarea)
      textarea.select()
      document.execCommand('copy')
      textarea.remove()
    },
    showToast(message) {
      const dark = this.theme === 'vs-dark'
      toast(message, {
        style: dark
          ? { background: 'var(--dark-article-background-color, var(--article-background-color, #1c1d1f))', border: '1px solid #484848', color: '#e0e0e0' }
          : { background: 'var(--light-article-background-color, var(--article-background-color, #fff))', border: '1px solid #d5d5d5', color: '#212529' }
      })
    },
    saveDraft() {
      localStorage.setItem(this.getDraftKey(), this.editor.getValue())
      this.showToast(this.editorText('draftSaved'))
      hideAllPoppers()
    },
    loadDraft() {
      const draft = localStorage.getItem(this.getDraftKey())
      if(draft !== null) {
        this.setValue(draft)
        this.showToast(this.editorText('draftLoaded'))
      } else this.showToast(this.editorText('noDraft'))
      hideAllPoppers()
    },
    async copyDocumentTitle() {
      const { title, rev } = this.getDocumentInfo()
      if(!title) return
      try {
        const actualTitle = await this.getActualDocumentTitle(title, rev)
        await this.copyText(actualTitle)
        this.showToast(this.editorText(rev ? 'copiedDocumentTitleWithRev' : 'copiedDocumentTitle', { title: actualTitle, rev }))
      } catch {
        this.showToast(this.editorText('copyFailed'))
      }
      hideAllPoppers()
    },
    async copyRaw() {
      try {
        await this.copyText(this.editor.getValue())
        const { title, rev } = this.getDocumentInfo()
        this.showToast(this.editorText(rev ? 'copiedRawContentWithRev' : 'copiedRawContent', { title, rev }))
      } catch {
        this.showToast(this.editorText('copyFailed'))
      }
      hideAllPoppers()
    },
    applyLinePrefix(prefix) {
      const model = this.editor.getModel()
      const lines = new Set()
      for(const selection of this.editor.getSelections()) {
        for(let line = selection.startLineNumber; line <= selection.endLineNumber; line++) lines.add(line)
      }
      const edits = [...lines].sort((left, right) => left - right).map(line => ({
        range: new this.monaco.Range(line, 1, line, model.getLineMaxColumn(line)),
        text: prefix + model.getLineContent(line)
      }))
      this.editor.executeEdits('toolbar', edits)
      this.editor.focus()
    },
    applyHeading(level) {
      const model = this.editor.getModel()
      const selections = this.editor.getSelections()
      const edits = selections.map(selection => {
        const startLine = selection.startLineNumber
        const endLine = selection.endLineNumber
        const range = new this.monaco.Range(startLine, 1, endLine, model.getLineMaxColumn(endLine))
        const lines = []
        for(let line = startLine; line <= endLine; line++) {
          const content = model.getLineContent(line).replace(/^=+\s*|\s*=+$/g, '').trim()
          lines.push(`${'='.repeat(level)} ${content || `${level}단계`} ${'='.repeat(level)}`)
        }
        return { range, text: lines.join('\n') }
      })
      this.editor.executeEdits('toolbar', edits)
      this.editor.focus()
    },
    toggleExpanded() {
      this.isExpanded = !this.isExpanded
      this.editor.layout()
      this.diffEditor?.layout()
    },
    async toggleCompare() {
      this.compareMode = !this.compareMode
      if(!this.compareMode) {
        this.diffEditor?.dispose()
        this.diffEditor = null
        return
      }

      await nextTick()
      const original = this.monaco.editor.createModel(this.originalContent, 'namumark')
      const modified = this.monaco.editor.createModel(this.editor.getValue(), 'namumark')
      this.diffEditor = markRaw(this.monaco.editor.createDiffEditor(this.$refs.compareDiv, {
        automaticLayout: true,
        minimap: { enabled: false },
        readOnly: true,
        renderSideBySide: true,
        theme: this.theme
      }))
      this.diffEditor.setModel({ original, modified })
    },
    updateStatus() {
      if(!this.editor) return
      const value = this.editor.getValue()
      const position = this.editor.getPosition()
      this.characterCount = value.length
      this.characterCountNoSpaces = value.replace(/\s/g, '').length
      this.lineCount = this.editor.getModel().getLineCount()
      this.cursorPosition = {
        line: position.lineNumber,
        column: position.column
      }
    }
  }
}
</script>
<style scoped>
section {
  background: var(--light-article-background-color, var(--article-background-color, #fff));
  color: var(--light-text-color, var(--text-color, #212529));
  display: flex;
  flex-direction: column;
  height: 34rem;
  width: 100%;
  border-radius: 4px;
}

.editor-toolbar,
.format-toolbar,
.editor-statusbar {
  align-items: center;
  background: var(--light-article-background-color, var(--article-background-color, #fff));
  border-color: #dfe1e2;
  display: flex;
  min-height: 2.35rem;
  padding: 0 .35rem;
}

.format-toolbar {
  border-bottom: 1px solid;
  flex: 0 0 auto;
  min-height: 2.65rem;
  position: relative;
  z-index: 2;
}

.toolbar-group { align-items: center; display: flex; gap: .1rem; }
.toolbar-button {
  border: 0;
  background: transparent !important;
  align-items: center;
  border-radius: 4px;
  color: inherit;
  display: inline-flex;
  gap: .35rem;
  height: 1.95rem;
  justify-content: center;
  min-width: 2rem;
  padding: 0 .4rem;
}
.toolbar-button:hover {
  background: var(--light-hover-background-color, var(--hover-background-color, #f0f0f0)) !important;
}
.toolbar-button svg { height: .95rem; width: .95rem; }
.toolbar-label { padding: 0 .55rem; }
.toolbar-icon { font-size: 1.05rem; line-height: 1; }
.toolbar-caret { font-size: .7rem; }
.italic { font-style: italic; }
.strike { text-decoration: line-through; }
.toolbar-divider { background: #dfe1e2; height: 19.5838px; margin: 0 .35rem; width: 2px; }
.toolbar-spacer { flex: 1; }
.editor-surface {
  flex: 1;
  font-family: D2Coding, Consolas, "나눔고딕코딩", "Courier New", monospace;
  min-height: 0;
  width: 100%;
  --vscode-editorCodeLens-lineHeight: 16px;
  --vscode-editorCodeLens-fontSize: 12px;
  --vscode-editorCodeLens-fontFeatureSettings: "liga" off, "calt" off;
}
.editor-statusbar { border-top: 1px solid #dfe1e2; color: var(--light-text-color, var(--text-color, #212529)); justify-content: flex-end; min-height: 1.8rem; font-size: .8rem; font-family: monospace; }
.editor-statusbar .toolbar-divider { height: 15px; margin: 0 .55rem; }
.muted { color: #6c757d; }
.is-expanded {
  height: 100vh;
  left: 0;
  overflow: hidden;
  top: 0;
  position: fixed;
  width: 100vw;
  z-index: 1000;
}
.is-expanded .format-toolbar {
  display: flex;
  flex: 0 0 2.65rem;
  min-height: 2.65rem;
  position: relative;
  visibility: visible;
  z-index: 100;
}
.compare-surface {
  min-height: 20rem;
}
:global(.editor-context-menu) {
  max-width: unset !important;
  padding: 12px;
}
:global(.editor-context-menu:has(.insert-menu)) {
  min-width: 20rem;
  width: 20rem;
}
:global(.editor-context-menu:has(.image-menu)) {
  width: 22rem;
}
:global(.editor-context-menu .insert-menu) {
  color: var(--light-text-color, var(--text-color, #212529));
  display: flex;
  flex-direction: column;
  gap: .45rem;
  min-width: 0;
}
:global(.editor-context-menu .insert-menu > label:not(.menu-check)) {
  margin: 0 !important;
  padding: 0 !important;
}
:global(.editor-context-menu .insert-menu .input),
:global(.editor-context-menu .insert-menu select) {
  box-sizing: border-box;
  min-width: 0;
  width: 100%;
}
:global(.editor-context-menu .insert-menu .menu-check) {
  align-items: center;
  display: inline-flex;
  min-width: 0;
  overflow: hidden;
  padding: 0 !important;
  white-space: nowrap;
}
:global(.editor-context-menu .insert-menu .menu-check span) {
  overflow: hidden;
  text-overflow: ellipsis;
}
:global(.editor-context-menu .insert-menu .field-grid) {
  align-items: center;
  display: grid;
  gap: .4rem;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
}
:global(.editor-context-menu .insert-menu .unit-field) {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 4.2rem;
  min-width: 0;
}
:global(.editor-context-menu .insert-menu .unit-field .input) {
  border-bottom-right-radius: 0;
  border-top-right-radius: 0;
}
:global(.editor-context-menu .insert-menu .unit-field select) {
  border-left: 0;
  border-bottom-left-radius: 0;
  border-top-left-radius: 0;
}
:global(.editor-context-menu .insert-menu > button) {
  align-self: flex-end;
  flex: 0 0 auto !important;
  margin: .55rem 0 0 !important;
  min-width: 4rem;
  padding: .3rem .75rem !important;
}
:global(.editor-context-menu .document-tools-menu) {
  box-sizing: border-box;
  display: flex !important;
  flex-direction: column;
  min-width: 12rem;
  overflow: hidden;
}
:global(.editor-context-menu .document-tools-menu button),
:global(.editor-context-menu .document-tools-menu a) {
  align-items: center;
  display: flex;
  gap: .5rem;
  justify-content: flex-start !important;
  margin: .125rem .25rem !important;
  width: auto;
  align-self: stretch;
}
:global(.editor-context-menu .document-tools-menu svg) {
  width: 1rem;
}
:global(.editor-context-menu .document-tools-menu button),
:global(.editor-context-menu .document-tools-menu a) {
  background: transparent !important;
  border: 0 !important;
  box-shadow: none !important;
  font-weight: 400;
  text-decoration: none !important;
}
:global(.editor-context-menu .document-tools-menu button:hover),
:global(.editor-context-menu .document-tools-menu a:hover) {
  background: var(--light-hover-background-color, #f0f0f0) !important;
}
:global(.editor-context-menu.theseed-dark-mode-context-menu .document-tools-menu button:hover),
:global(.editor-context-menu.theseed-dark-mode-context-menu .document-tools-menu a:hover) {
  background: var(--dark-hover-background-color, #2d2f34) !important;
  color: var(--dark-text-color, #e0e0e0) !important;
}
:global(.editor-context-menu.heading-menu) {
  min-width: 14rem;
  width: 14rem;
}
:global(.editor-context-menu .heading-option) {
  align-items: center;
  display: flex;
  justify-content: space-between !important;
  width: 100%;
}
:global(.editor-context-menu .heading-option),
:global(.editor-context-menu .font-size-option) {
  background: transparent !important;
  border: 0 !important;
  box-shadow: none !important;
  font-weight: 400;
  line-height: 1.5;
  text-decoration: none !important;
}
:global(.editor-context-menu .heading-option:hover),
:global(.editor-context-menu .font-size-option:hover) {
  background: var(--light-hover-background-color, #f0f0f0) !important;
  color: inherit !important;
}
:global(.editor-context-menu .heading-preview) {
  background: var(--light-article-background-color, #f0f0f0);
  border-radius: 4px;
  color: #777;
  font-size: .75rem;
  padding: .2rem .4rem;
}
:global(.editor-context-menu .font-size-option) {
  align-items: center;
  display: flex;
  justify-content: space-between !important;
  width: 100%;
}
:global(.editor-context-menu .font-size-preview) {
  color: #777;
  padding-left: .75rem;
}
:global(.editor-context-menu .font-size-preview--minus2) { font-size: .8rem; }
:global(.editor-context-menu .font-size-preview--minus1) { font-size: .9rem; }
:global(.editor-context-menu .font-size-preview--1) { font-size: 1.1rem; }
:global(.editor-context-menu .font-size-preview--2) { font-size: 1.2rem; }
:global(.editor-context-menu .color-field) {
  display: flex;
}
:global(.editor-context-menu .color-picker-input) {
  border-bottom-right-radius: 0;
  border-right-width: 0;
  border-top-right-radius: 0;
  height: 2.1rem;
  padding: 0;
  width: 1.9rem;
}
:global(.editor-context-menu .image-background-field .color-picker-input) {
  width: 2.2rem;
}
:global(.editor-context-menu .color-text-input) {
  border-bottom-left-radius: 0;
  border-top-left-radius: 0;
}
:global(.editor-context-menu .insert-actions) {
  display: flex;
  justify-content: flex-end;
  margin: .75rem 0 0;
}
:global(.editor-context-menu .external-sites) {
  font-size: .85rem;
}
:global(.editor-context-menu .external-sites-list) {
  padding: 0 0 0 1.5rem;
}
:global(.editor-context-menu .video-menu ul) {
  margin: .15rem 0 0;
  padding-left: 1.2rem;
}
:global(.editor-context-menu .image-menu .input[type=color]) {
  min-height: 1.95rem;
  padding: .1rem;
}
:global(.editor-context-menu.theseed-dark-mode-context-menu .insert-menu) {
  background: var(--dark-article-background-color, #1c1d1f);
  color: var(--dark-text-color, var(--text-color, #e0e0e0));
}
.theseed-dark-mode {
  background: var(--dark-article-background-color, var(--article-background-color, #1c1d1f));
  border-color: #5c5c5c;
  color: var(--dark-text-color, var(--text-color, #e0e0e0));
}
.theseed-dark-mode .editor-toolbar,
.theseed-dark-mode .format-toolbar,
.theseed-dark-mode .editor-statusbar {
  background: var(--dark-article-background-color, var(--article-background-color, #2e2e2e));
  border-color: #5c5c5c;
}
.theseed-dark-mode .toolbar-divider { background: #5c5c5c; }
.theseed-dark-mode .editor-statusbar { color: var(--dark-text-color, var(--text-color, #e0e0e0)); }
.theseed-dark-mode .muted { color: #888; }
@media (max-width: 640px) {
  .toolbar-label span:not(.toolbar-icon):not(.toolbar-caret) { display: none; }
  .format-toolbar { overflow-x: auto; }
  .editor-statusbar { gap: .5rem; font-size: .72rem; }
}
.theseed-dark-mode .toolbar-button:hover {
  background: var(--dark-hover-background-color, var(--hover-background-color, #2d2e2f)) !important;
}
:global(.editor-context-menu.theseed-dark-mode-context-menu .heading-preview) {
  background: var(--dark-article-background-color, #2d2e2f);
  color: #aaa;
}
:global(.editor-context-menu.theseed-dark-mode-context-menu .heading-option:hover),
:global(.editor-context-menu.theseed-dark-mode-context-menu .font-size-option:hover) {
  background: var(--dark-hover-background-color, #2d2e2f) !important;
}
</style>