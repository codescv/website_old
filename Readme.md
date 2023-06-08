# Default theme location

```Bash
cd $(bundle info --path minima)
```

# Convert notebook to html:
```Bash
cd _scripts && python nb2post.py
```

# Import from notion
1. Export markdown from notion
2. copy notion/blog-name directory to images/
3. Fix relative image paths
```bash
cat notion/blog-name.md| sed '/\!\[.*\]/s/(\(.*\))/(\/images\/\1)/g' > _posts/xxxx-xx-xx-blog-name.md
```
4. edit blog markdown to add front matter, fix minor issues etc.
