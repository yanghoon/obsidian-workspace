### [How to set a white background partially for a diagram in a note?](https://forum.obsidian.md/t/how-to-set-a-white-background-partially-for-a-diagram-in-a-note/58497)

```css
.image-embed.is-loaded img[alt*="white"] { background-color: white; }
```

```markdown
![[3D.png|white]]
```