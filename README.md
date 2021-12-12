# log4j-gitlab-search
Script to search for a word (i.e. log4j) in all gitlab repos (free version doesn't allow bulk searchs)

## Prerequisites
- apt install html-xml-utils 

## Code
```bash
#!/bin/bash


privatetoken="glpat-xxx"
token="xxx"
word=log4j
gitlaburl="gitlab.orgao.gov.br"

# set privatetoken/token/word before
# get all project_id/group_id and path_with_namespace using gitlab API and stores them in p.txt file
rm p.txt; for((i=1;i<100;i++)); do echo $i; saida=$(curl -k -s --header "PRIVATE-TOKEN: ${privatetoken}" "https://${gitlaburl}/api/v4/projects/?private=true&per_page=100&page=${i}" | jq '.[] | "\(.id)|\(.namespace.id)|\(.path_with_namespace)"' | tr -d '"'); num=$(echo -e $saida| sed 's/ /\n/g' | wc -l); echo $num; if [[ $num -eq 1 ]]; then break; fi; echo -e $saida | sed 's/ /\n/g' >> p.txt; done; echo acabou;

# walk through all projects within p.txt searching for the word 
cat p.txt | while read l; do echo $l; IFS='|' read -r -a array <<< $l; url="https://${gitlaburl}/search?project_id=${array[0]}&group_id=${array[1]}&search=${word}"; echo $url; curl -k -s --header "Cookie: _gitlab_session=${token}"  ${url} | hxnormalize -x | tr -d '\n' |  hxselect  'pre.code' -c | sed 's/<span class[^>]\+>//g' | sed 's/<\/span>//g' | hxselect code -c -s '\n' | cat -n |  hxunent; done | tee log4j.txt

```
