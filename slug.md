#  SEO友好的URL结构（Slug实现）

How would you reference this object with a URL, with a meaningful name? You could use Article.id so the URL would look like this:

```
www.example.com/article/23
```

Or, you could reference the title like so:

```
www.example.com/article/The 46 Year Old Virgin
```

Problem is, spaces aren't valid in URLs, they need to be replaced by `%20` which is ugly, making it the following:

```
www.example.com/article/The%2046%20Year%20Old%20Virgin
```

That's not solving our meaningful URL. Wouldn't this be better:

```
www.example.com/article/the-46-year-old-virgin
```

That's a slug. **`the-46-year-old-virgin`**. All letters are downcased and spaces are replaced by hyphens `-`. See the URL of this very webpage for an example!

 

```
 $slug = url_title($this->input->post('title'), 'dash', TRUE);
```

用于将字符串 中的所有空格替换成连接符（-），并将所有字符转换为小写。 这样其实就生成了一个 slug ，可以很好的用于创建 URI 。

 

http://stackoverflow.com/questions/427102/what-is-a-slug-in-django

http://www.sjyhome.com/wordpress/wp-slug.html