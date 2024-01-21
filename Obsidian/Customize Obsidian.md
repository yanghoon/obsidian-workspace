### [How to set a white background partially for a diagram in a note?](https://forum.obsidian.md/t/how-to-set-a-white-background-partially-for-a-diagram-in-a-note/58497)

```css
.image-embed.is-loaded img[alt*="white"] { background-color: white; }
```

```markdown
![[my_image.png|white]]
```

### [Obsidian uses Prism for syntax highlighting](https://help.obsidian.md/Editing+and+formatting/Basic+formatting+syntax#Code+blocks)
* [Supported languages](https://prismjs.com/#supported-languages)