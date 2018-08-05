# THe-Military-Grind

## How to Deploy?

```
cd ./the-military-grind
rm -r docs/
jekyll build # This should create a folder called _site.
mv _site docs # Change the name of the _site folder to "docs".
git add file_names
git commit -m "your commit message describing what you did"
git push origin master
```
