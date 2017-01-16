# imSitu
Suport files for the imSitu dataset. 

Annotation files and evaluation scripts of imsitu. 

## imsitu_space.json
A json file defining the metadata used for imSitu. 

Each verb in imSitu is mapped to a FrameNet frame, for example, 'swimming' is mapped to Self_motion, and each of its roles is mapped to a role associated with that frame, for example, the imSitu role 'agent' for swimming is mapped to the framenet role 'self_mover'. Each noun in space corresponds to a synset from Wordnet.
 
imsitu_space.json contians these mappings, as well as definitions of verbs, roles and an abstract sentence presented to turkers to help them understand the roles quickly. Furthermore, it contains definitions of nouns and glosses for those nouns that were presented to turkers.
 
```
import json

imsitu = json.load(open("imsitu_space.json"))

nouns = imsitu["nouns"]
verbs = imsitu["verbs"]

print verbs["swimming"]

#{u'roles': {u'place': {u'framenet': u'place', u'def': u'The location where the swim event is happening'}, 
#u'agent': {u'framenet': u'self_mover', u'def': u'The entity doing the swim action'}}, 
#u'framenet': u'Self_motion',u'abstract': u'the AGENT swims in a PLACE.', u'order': [u'agent', u'place'], u'def': u'propel oneself through water by bodily movement'}

print nouns["n02129165"]

#{u'def': u'large gregarious predatory feline of Africa and India having a tawny coat with a shaggy mane in the male',
# u'gloss': [u'lion', u'king of beasts', u'Panthera leo']}

```
## train.json, dev.json, test.json

These files contain annotations for the train/dev/test set. Each image in the imSitu dataset is annotated with three frames corresponding to one verb.  

```
import json
train = json.load(open("train.json"))

train['clinging_250.jpg']
#{u'frames': [{u'agent': u'n01882714', u'clungto': u'n05563770', u'place': u''},
#  {u'agent': u'n01882714', u'clungto': u'n05563770', u'place': u''},
#  {u'agent': u'n01882714', u'clungto': u'n00007846', u'place': u''}],
# u'verb': u'clinging'}

```

## images
Original downloaded images can be found here (34G) :

https://s3.amazonaws.com/my89-frame-annotation/public/of500_images_resized.tar

and images resized to 256x256 here (3.7G):

https://s3.amazonaws.com/my89-frame-annotation/public/of500_images_resized.tar
