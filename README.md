# Data Masking

### Problem: Mask the personal information from the email 
Instead of going with state-of-the-art models(Transformers) and relying 100% on probability. <br>
I decided to go with <b>  SPACY (Rule-based + ML-based)</b>, To decrease the chances of error. <br><br>

<I> 💙 And wanted to share the base version of my code with you guys</i>.

#### End Result🎄

![image](https://github.com/LLama2-Ai/spacyCustomNER/assets/142317270/587d58f7-ac4b-40fe-9167-6c2db7375cc7)

### Requirements.
<ul>
  <li> <b>Data:</b> to capture the names correctly, As names cannot be done with the rule-based approach </li>
  <li> <b> Regular expressions:</b> to fetch phoneNo,Dates,Email addresses, Card details  </li>
</ul>

## Table of Contents

- [Installation & Imports](#installation)
- [Data Loading](#usage)
- [Data Creation](#data)
- [Training](#training)
- [Preparing Rule-Based](#training)
- [Loading the model](#rulebased)
- [Results](#Results)

## Installation

<ul>
  <li>! pip install spacy</li>
  <li>import spacy</li>
  <li>import re</li>
  <li>from spacy.language import Language</li>
</ul>


## Data Load
<p> 
Bellow code will read all the text files listed in the names directory. And creates a data frame of names exist in different languages </p>

```
def read_data():
    path = r'names' # use your path
    res=os.listdir(path)
    dataList=[]
    for i in res:
        dataList.append(pd.read_csv(path+"\\"+i,sep='\t+',header=None,engine='python'))
    fullDs=pd.concat(dataList,ignore_index=True,axis=0)
    fullDs.rename(columns={0:'Name'},inplace=True)
    return fullDs
```



## Data Creation

<table>
  <tr><td>Sample</td> <td>Output from Function</td></tr>
  <tr>
    <td>
  From: <b>name </b> <name@tevera.com>
  Sent: Friday, September 22, 2023 8:46 AM
  To: <b>name </b> <name@kandi.com>; <b>name </b> <name@kandi.com>; IT Claims Data Model name <name@tevera.com>; name name name <name@tevera.com>
  Cc: <b>name </b><name@kandi.com>; <b>name </b><name@kandi.com>
  Subject: [EXTERNAL] RE: Old GRP's on Load Env    
    </td>
    <td>
      From: <b>InputFromArray</b> <name@tevera.com>
Sent: Friday, September 22, 2023 8:46 AM
To: <b>InputFromArray</b> <name@kandi.com>; <b>InputFromArray</b>  <name@kandi.com>; <b>InputFromArray</b> <InputFromArray@tevera.com>;<b>InputFromArray</b>  <name@tevera.com>
Cc: <b>InputFromArray</b> <name@kandi.com>; <b>InputFromArray</b>  <name@kandi.com>
Subject: [EXTERNAL] RE: Old GRP's on Load Env
    </td>
  </tr>
</table>


<p> 
Bellow code will generate data for training using templates. and a list of some values you want to replace in the template one by one.
like in my case 
  the <b>name</b>  word will be replaced with the names or any attribute provided in the list.  
</p>


<ul>
  <li><b>fullDs:</b> DataFrame of Names</li>
  <li><b>Email Templates:</b> In my case email you can use any template</li>
  <li><b>LowerBound:</b> Min count of template-array, function will choose a random template in each iteration</li>
  <li><b>UpperBound:</b> Max count of template-array, function will choose a random template in each iteration</li>
  <li><b>max:</b> Number of samples you need</li>
</ul>

```
def prepareData(fullDs,email_templates,lowerBound,upperBound,max=0):
    emails=[]
    listNames=fullDs.Name.tolist()
    listLen=len(listNames)
    for i in range(listLen if max==0 else max):
        template=email_templates[r.randint(lowerBound,upperBound)]
        spans=[]
        stri=''
        for index,str in enumerate(template.split('name')):
            stri+=str
            spans.append((len(stri),len(stri+listNames[index]),'NAME'))
            stri+=listNames[index]
        emails.append((stri,{'entities':spans}))
    return emails

fullDs=read_data()
emailsTrain=prepareData(fullDs,email_templates_train,0,4)
emailsTest=prepareData(fullDs,email_templates_test,0,1,500)
print(len(emailsTrain),len(emailsTest))
```

## Raw Data to .Spacy Format

``` 
def saveForSpacy(emails,fileName):
    db=DocBin()
    for text, annot in emails:
        #print(annot['entities'])
        doc=nlp(text)
        ents=[]
        
        for start,end,label in annot['entities']:
            span=doc.char_span(start, end, label=label,alignment_mode="strict")
            if span is None:
                pass
            else:
                ents.append(span)

        doc.ents=ents
        db.add(doc)
    db.to_disk(''+fileName+'.spacy')

saveForSpacy(emailsTrain,'train')
saveForSpacy(emailsTest,'test')
```
## Config 
```
! python -m spacy init config config.cfg --lang en --pipeline ner --optimize accuracy 
```
## Traning
```
! python -m spacy train config.cfg --output ./ --paths.train ./train.spacy --paths.dev ./test.spacy
```
<p> After training two files will be generated </p>
<ol><li>model-best</li><li>model-last</li></ol>

## Rule based 

## ModelLoading
You can load any model.
``` nlp=spacy.load('model-last') ```
## Results
