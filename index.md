## Data integration based on time stamp using SSIS

To performing a data integration process between two database systems, there will be two methods to do this:
The first method is to do a full data integrastion. Thats means the whole data in database A will be copied and send to database B. Normally, thiis kind of integration process is used for one time only at themost first database integration process.

The second method is a partial data intefgration thats operate between two database systems depending on a condition. However, in this method, only a pointed records inside the database A will be integrated with database B. As an example, the daily fingerprint records for the time attendance system thats required to integrate the daily records only. These daily records can be assigned by the system regarding one of these factors:

The first factor is

You can use the [editor on GitHub](https://github.com/mbmasadeh/TimeStampDataMigration/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/mbmasadeh/TimeStampDataMigration/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
