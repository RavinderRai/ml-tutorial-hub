# Named Entity Recognition with Spacy and OpenAI

In this quick tutorial we will go over an example of how to use spacy's new LLM capabilities, where it leverages OpenAI to make NLP tasks super simple. Here we will focus on an NER task, which means we will specify entities in text that we want to extract, and try to identify those. You can find the documentation here:

 - [Spacy LLM Full Documentation](https://github.com/explosion/spacy-llm)
 - [Spacy LLM NER Documentation](https://github.com/explosion/spacy-llm/tree/main/usage_examples/ner_v3_openai)

but we will go over a different example below, where we extract Superhero names and their gadgets and weapons from text.

## Getting Started

First we need to import the necessary packages. The only noteworthy one is the spacy_llm package which you can install with `pip install spacy-llm`. Then create a .env file with your OpenAI API key.


```python
import os
from dotenv import load_dotenv
from spacy_llm.util import assemble

load_dotenv()

openai_api_key = os.getenv('OPENAI_API_KEY')
```

Now we can build our ner object with the assemble function


```python
ner = assemble("config.cfg", overrides={"paths.examples": "examples.json"})
```

The next steps are straightforward, but that's because a lot of the work is done in the config.cfg and examples.json files, so we'll go over them now.

## Config File

Create a config.cfg file in the same directory. We have it here already so we'll print it out and you can copy and paste it. 


```python
# Display the contents in a more readable manner with a scrollbar
from IPython.display import display, HTML

with open('config.cfg', 'r') as file:
    config_contents = file.read()

html_content = f'<div style="height: 400px; overflow: auto; border: 1px solid #ddd; padding: 10px;"><pre>{config_contents}</pre></div>'
display(HTML(html_content))
```


<div style="border: 1px solid #ddd; padding: 10px;"><pre>[paths]
examples = null

[nlp]
lang = "en"
pipeline = ["llm"]

[components]

[components.llm]
factory = "llm"

[components.llm.task]
@llm_tasks = "spacy.NER.v3"
labels = ["SUPERHERO", "GADGET", "WEAPON"]
description = Entities are the names Superheroes,
    their gadgets, and their weapons.
    Adjectives, verbs, adverbs are not entities.
    Pronouns are not entities.

[components.llm.task.label_definitions]
SUPERHERO = "Known superheroes from comic books or movies, e.g. Batman, Superman, Wonder Woman, Spider-Man"
GADGET = "Helpful tools that might be well known or associated with superheroes, e.g. Batman's Grapple Hook, Wonder Woman's Invisible Jet, Spider-Man's Web Wings."
WEAPON = "Any kind of item that is used as a weapon, e.g. Batman's Batarang, Spider-Man's Web Shooters, Wonder Woman's Tiara, "

[components.llm.task.examples]
@misc = "spacy.FewShotReader.v1"
path = "${paths.examples}"

[components.llm.model]
@llm_models = "spacy.GPT-3-5.v3"</pre></div>


There's only a few components that are important for our use case. The first is the [components.llm.task] part, where you need to define your labels or entities that you want to identify. Then give a short description on the general context of what you're looking for. 

Then in the [components.llm.task.label_definitions] component, you'll really give solid definitions of each label. You can come back and play around with these if your results aren't great, but I find being clear yet concise to work just fine.

Finally, the [components.llm.model] where where you can try different versions of each OpenAI model. You can find the supported models available here: [Spacy LLM Models](https://spacy.io/api/large-language-models#models).

## Examples File

Next is the examples file. This is really important as this NER method relies on the few-shot technique and chain-of-thought reasoning, so you're examples could make or break your program. So to make this work create an examples.json file with your own examples. From the documentation, it mentions having negative examples improves performance, so consider that when making your own, and we'll show that here to.

Instead of loading the examples.json file, below is a way to write it in a notebook and then create the file here, which might be more convenient. Note the `==NONE==` labels, which are to identify false examples. You will encounter a user warning later saying there's a label not recognized, but that's just due to this `==NONE==` label so you can safely ignore it.


```python
data = [
    {
      "text": "Batman can take on any villain thanks to his intellect in combination with his many tools in his utility belt.",
      "spans": [
        {
            "text": "Batman",
            "is_entity": True,
            "label": "SUPERHERO",
            "reason": "Batman is a popular superhero that originated from DC comic books"
        },
        {
          "text": "intellect",
          "is_entity": False,
          "label": "==NONE==",
          "reason": "is an attribute of Batman's character, but not exactly a weapon, nor a gadget"
        },
        {
          "text": "Utility Belt",
          "is_entity": True,
          "label": "GADGET",
          "reason": "is a gadget that is commonly associated with Batman"
        }
      ]
    },
    {
      "text": "Even with Mjolnir and super strength, Thor could not beat Superman in a one-on-one fight during the Marvel and DC crossover comic book",
      "spans": [
        {
          "text": "Mjolnir",
          "is_entity": True,
          "label": "WEAPON",
          "reason": "is a popular weapon (hammer) that belongs to the Superhero Thor"
        },
        {
            "text": "super strength",
            "is_entity": False,
            "label": "==NONE==",
            "reason": "super strenth is a super power that belongs to many super heros, it is not specifically a weapon"
        },
        {
            "text": "Thor",
            "is_entity": True,
            "label": "SUPERHERO",
            "reason": "Thor is a popular superhero that originated from Marvel comic books, and Norse mythology before that"
        },
        {
            "text": "Superman",
            "is_entity": True,
            "label": "SUPERHERO",
            "reason": "Superman is a popular superhero that originated from DC comic books"
        }
      ]
    }
  ]
```


```python
import json

# Write to a JSON file
with open('examples.json', 'w') as f:
    json.dump(data, f, indent=4)
```

## Identifying Entities

With those files ready, now you can test it out. Remember we are using superheroes as our general topic, so our example text is: "Both Superman and Wonder Woman lifted Mjolnir once before, because it was believed to be safe in Superman's hands so an exception was made, and Wonder Woman because she was actually worthy."

From this, we'd expect to see entities Superman, Wonder Woman, and Mjolnir, with the labels SUPERHERO, SUPERHERO, and WEAPON, respectively.


```python
example_text = "Both Superman and Wonder Woman lifted Mjolnir once before, because it was " \
               "believed to be safe in Superman's hands so an exception was made, and " \
               "Wonder Woman because she was actually worthy."

doc = ner(example_text)
```

To get the entities with their labels, you need to iterate over doc.ents, which we do below with a list comprehension.


```python
entities = [(ent.text, ent.label_) for ent in doc.ents]
entities
```




    [('Superman', 'SUPERHERO'),
     ('Wonder Woman', 'SUPERHERO'),
     ('Mjolnir', 'WEAPON')]



There we go! Works perfectly as expected. Now let's put it all together in a little function.


```python
def extract_entities(text, config_file, examples_file):
    #build our ner object using our configuration and examples
    ner = assemble(config_file, overrides={"paths.examples": examples_file})

    #input our text
    doc = ner(text)

    #get our extracted entities
    entities = [(ent.text, ent.label_) for ent in doc.ents]

    return entities
```

Let's finish off with a few more examples, or rather a small dataset of text samples.


```python
superhero_text_examples = [
    "Both Superman and Wonder Woman lifted Mjolnir once before, because it was " \
    "believed to be safe in Superman's hands so an exception was made, and " \
    "Wonder Woman because she was actually worthy.",

    "Batman used his grappling hook to swing across the rooftops of Gotham City, " \
    "and then threw a batarang to disarm the Joker, who was wielding a crowbar.",

    "The Flash used his super speed to rescue civilians from a burning building, " \
    "while Green Lantern created a protective barrier to contain the fire.",
]
```


```python
for superhero_text_sample in superhero_text_examples:
    entities = extract_entities(superhero_text_sample, "config.cfg", "examples.json")
    print(entities)
```

    [('Superman', 'SUPERHERO'), ('Wonder Woman', 'SUPERHERO'), ('Mjolnir', 'WEAPON')]
    [('Batman', 'SUPERHERO'), ('grappling hook', 'GADGET'), ('batarang', 'WEAPON'), ('crowbar', 'WEAPON')]
    [('The Flash', 'SUPERHERO'), ('Green Lantern', 'SUPERHERO')]
    
