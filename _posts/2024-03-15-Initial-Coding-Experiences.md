# Initial Coding Experiences
Here you will find some code that I have written during the introductory year of my BA for the courses *Programming Techniques in Computational Linguistics 1 & 2*: 

* ## [Processing a CSV File](https://za-aftab.github.io/posts/Beginning/#processing-a-csv-file-1)
* ## [XML & JSON](https://za-aftab.github.io/posts/Beginning/#xml--json-1)
* ## [Encodings](https://za-aftab.github.io/posts/Beginning/#encodings-1)
* ## [Recursive Algorithms](https://za-aftab.github.io/posts/Beginning/#recursive-algorithms-1)

## Processing a CSV File
In this exercise, I implemented the module processing.py to process natural language data, namely dad jokes.
The processing involves sentence splitting, tokenizing, filtering profanity and printing out readable output.

**Data Sources:**
* ```dadjokes_samples.txt```: https://www.kaggle.com/oktayozturk010/reddit-dad-jokes
* ```profanities.txt```: https://www.freewebheaders.com/full-list-of-bad-words-banned-by-google/

```python
from typing import List, Tuple
import re




def split_into_sentences(post_str: str) -> List[str]:
	"""Split text into sentences"""
	post_str = post_str.rstrip('\n')
	output = re.split(r' ?([^.!?\n]+[.?!]{1,3}|\n)', post_str)
	return [sent.lstrip() for sent in output if sent]


def tokenize(sentences_str: List) -> List[List[str]]:
	"""Tokenize all the words in the sentence"""

	output = [re.findall(rf'[\w]+|\'|’|"|\?|¿|\.|\n|,|!|-|–|\(|\)|…', sentence) for sentence in sentences_str if sentence]
	return output


def filter_profanity(tokenized: List[List[str]], filename: str) -> Tuple[List[List[str]], int]:
	"""Filter out all the profanity"""

	output = []

	# Count number of profanities
	num_profanities = 0

	# Read in profanity file and store profanities in a list
	with open(filename, "r", encoding="utf-8") as file:
		profanities = file.read().split("\n")

	lst = []
	# Go over every sentence
	for sentence in tokenized:
		for prof in profanities:
			for token in sentence:
				# Check if there is a profanity
				if token.lower() in profanities or prof in token.lower():
					num_profanities += 1
					# Censor
					sentence[sentence.index(token)] = len(token) * '#'

		# Avoid appending empty lists
		if sentence:
			lst.append(sentence)
	return(lst, num_profanities)


def pretty_print(processed: List[List[str]]) -> None:
    out = ''
    stars = '***'
    for joke in processed[::2]:
        for sent in joke[::2]:
            for word in sent:
                if not word == '?' or word == '!' or word == '.' or word == '(':
                    out += ' ' + word
                else:
                    out += word

        output = '_' * 160 + 2 * "\n"
        print(f'{output}{stars} {out} {stars}')
        out = ''
    return None


def main():
	with open("dadjokes_samples.txt", "r", encoding="utf-8") as file:
		for line in file:
			sentence = split_into_sentences(line)
			# print(sentence)
			tokenized_sentence = tokenize(sentence)
			# print(tokenized_sentence)
			filtered_sentence = filter_profanity(tokenized_sentence, 'profanities.txt')
			# print(filtered_sentence)
			pretty_print(filtered_sentence[0])
			print()


if __name__ == '__main__':
	main()
```

## XML & JSON
In this task the Joke and JokeGenerator classes are extended to save their attributes
in XML or JSON representation. In addition, I adapted the ```make_jokes_objects(self)``` method from the JokeGenerator class so that it will also
accept files containing jokes in a JSON representation. The method should still return a list containing all the jokes from
the file as Joke objects, so that no changes outside the function have to be made.

**Data Sources:**
* ```dadjokes_samples.txt```: https://www.kaggle.com/oktayozturk010/reddit-dad-jokes
* ```profanities.txt```: https://www.freewebheaders.com/full-list-of-bad-words-banned-by-google/

```python
import time
from typing import List, Tuple, Dict
import re
import random
import csv
from lxml import etree
import json


class Joke:
    """The Joke object contains the joke, and some metadata on that joke. One can compare the jokes by upvotes"""
    def __init__(self, raw_joke):
        self.raw_joke = raw_joke
        self.author = self.raw_joke[0]
        self.link = self.raw_joke[1]
        self.joke = self.raw_joke[2]
        self.rating = int(self.raw_joke[3])
        self.time = self.raw_joke[4]

        self.sentences_joke = self.split_into_sentences()
        self.tokenized_joke = self._tokenize()
        self.filtered_joke = self.filter_profanity()[0]
        self.num_profanities = self.filter_profanity()[1]

        # Because of _get_xml_repr-function being a property decorator in this script, the brackets are left out.
        self.xml_rep = self._get_xml_repr
        self.json_rep = self._get_json_repr()

    def split_into_sentences(self) -> List[str]:
        """Split text into sentences"""
        output = re.findall(r' ?([^.!?\n]+[.?!]*|\n)', self.joke)
        return output

    def _tokenize(self) -> List[List[str]]:
        """Tokenize all the words in the sentences"""
        output = []
        for sentence in self.sentences_joke:
            tokenized_sentence = re.findall(r'([\w\']+|\?|\.|\n|,|!)', sentence)
            output.append(tokenized_sentence)
        return output

    def filter_profanity(self, filename="profanities.txt") -> Tuple[List[List[str]], int]:
        """Filter out all the profanity"""

        output = []

        # Count number of profanities
        num_profanities = 0

        # Read in profanity file
        with open(filename, "r")as file:
            profanities = file.read().split("\n")

        for sentence in self.tokenized_joke:
            no_profanity = True
            text_sentence = " ".join(sentence)
            for profanity in profanities:

                # Check if there is profanity in the sentence
                if profanity in text_sentence:
                    profanity_in_text = True
                else:
                    profanity_in_text = False

                while profanity_in_text:
                    num_profanities += 1
                    no_profanity = False

                    # Find the index of the profanity
                    index = text_sentence.index(profanity)
                    front = text_sentence[:index - 1]

                    # Find the words that need to be replaced
                    num_words_before_profanity = len(front.split(" "))
                    num_profanity_words = len(profanity.split(" "))
                    profanity_in_sentence = sentence[num_words_before_profanity: num_words_before_profanity + num_profanity_words]

                    # Replace the profanity with '#'
                    replacement = ["#" * len(word) for word in profanity_in_sentence]

                    # Construct new sentence composed of the parts with and without profanity
                    new_sent = []
                    new_sent.extend(sentence[:num_words_before_profanity])
                    new_sent.extend(replacement)
                    new_sent.extend(sentence[num_words_before_profanity + len(replacement):])
                    text_sentence = " ".join(new_sent)
                    sentence = new_sent

                    # Check if there is still profanity in the sentence
                    if profanity in text_sentence:
                        profanity_in_text = True

                    else:
                        profanity_in_text = False
                        output.append(new_sent)

            # Add sentence immediately if there are no profanities in the sentence
            if no_profanity:
                output.append(sentence)
        return output, num_profanities

    def tell_joke(self):
        if len(self.filtered_joke) > 1:
            build_up = self.filtered_joke[:-1]
            punch_line = self.filtered_joke[-1:]

            print(self.pretty_print(build_up))
            time.sleep(1)
            print(self.pretty_print(punch_line))
        else:
            print(self.pretty_print(self.filtered_joke))

    @staticmethod
    def pretty_print(joke) -> str:
        """Print in a humanly readable way"""
        output = ""
        for sentence in joke:
            output += " ".join(sentence) + " "
        return output

    # A property decorator provides methods for removing, changing and accessing the attributes of an object.
    @property
    def _get_xml_repr(self) -> etree.Element:
        """Get the xml representation of the Joke with all its attributes as nodes"""

        # 'root' is the top element that contains all other (sub-) elements. In this case we name it 'joke'.
        root = etree.Element("joke")
        # With etree.SubElement(root, 'text') a sub-element, text, of 'root' is defined.
        text = etree.SubElement(root, "text")
        # In this sub-element the information of self.joke is stored/saved through text.text
        text.text = self.joke
        author = etree.SubElement(root, "author")
        author.text = self.author
        link = etree.SubElement(root, "link")
        link.text = self.link
        rating = etree.SubElement(root, "rating")
        # If the information isn't saved into the sub-element as a string an error occurs.
        rating.text = str(self.rating)
        time = etree.SubElement(root, "time")
        time.text = self.time
        profanity_score = etree.SubElement(root, "profanity_score")
        profanity_score.text = str(self.num_profanities)

        # The variable 'root' returns the entire tree structure.
        return root

    def _get_json_repr(self) -> Dict:
        """Get the json representation of the Joke with all its attributes as keys and values"""

        # Firstly, a dictionary (json_dict) is defined where all the keys with their values are stored in.
        json_dict = {}

        # The following part stores the values to their corresponding keys.
        json_dict["author"] = self.author
        json_dict["link"] = self.link
        json_dict["text"] = self.joke
        json_dict["rating"] = self.rating
        json_dict["time"] = self.time
        json_dict["profanity_score"] = self.num_profanities

        # The dictionary is lastly returned.
        return json_dict

    def __repr__(self):
        """Allows for printing"""
        return self.pretty_print(self.filtered_joke)

    def __eq__(self, other):
        """Equal rating"""
        return self.rating == other.rating

    def __lt__(self, other):
        """less than rating"""
        return self.rating > other.rating

    def __gt__(self, other):
        """greater than rating"""
        return self.rating < other.rating

    def __le__(self, other):
        """less than or equal rating"""
        return self.rating >= other.rating

    def __ge__(self, other):
        """greater than or equal rating"""
        return self.rating <= other.rating


class JokeGenerator:
    def __init__(self, filename="reddit_dadjokes.csv"):
        self.filename = filename
        self.jokes = self.make_jokes_objects()

    def make_jokes_objects(self):
        """This function is adapted to also accept files containing jokes in a JSON representation."""

        with open(self.filename, "r", encoding="utf-8") as file:
            # This if-statement checks if the filename is a json-file. If not, a csv-file is assumed.
            if str(self.filename).endswith("json"):
                # json.load() is a method that takes file objects and returns json objects.
                # This lets the code parse the file, in this case the json object is saved to the variable 'jokes_dict'.
                jokes_dict = json.load(file)
                jokes_list = []

                # The following for-loop goes through the values of jokes_dict and returns a listed list, jokes_list.
                for value in jokes_dict.values():
                    # To save each joke separately to jokes_list, the following list is redefined with every iteration.
                    new_joke = []

                    # The values of the dictionary are stored into their corresponding variables.
                    author = value["author"]
                    link = value["link"]
                    text = value["text"]
                    rating = value["rating"]
                    time = value["time"]

                    # These variables are then appended to the new_joke-list.
                    new_joke.append(author)
                    new_joke.append(link)
                    new_joke.append(text)
                    new_joke.append(rating)
                    new_joke.append(time)

                    # The 'new_joke'-list is appended to jokes_list.
                    jokes_list.append(Joke(new_joke))

                return jokes_list

            lines = csv.reader(file, delimiter=',')
            jokes = [Joke(row) for row in lines]

            return jokes

    def generate_jokes(self):
        for joke in self.jokes:
            if len(joke.filtered_joke) > 1:
                joke.tell_joke()
            time.sleep(10)

    def random_joke(self):
        joke = random.sample(self.jokes, 1)[0]
        joke.tell_joke()

    def save_jokes_xml(self, outfile: str) -> None:
        """Saves all the jokes of the Generator in their xml representation to the outfile"""

        # Firstly, a supra element, 'jokes', is defined where each individual joke will be saved into later.
        jokes = etree.Element("jokes")

        # The following for-loop goes through each object in self.jokes, changes them to xml-nodes and
        # appends each node to 'jokes'.
        for joke in self.jokes:
            jokes.append(joke.xml_rep)

        # etree.tostring() returns a byte-string of the jokes.
        # pretty_print=True enables XML-documents to be easily read by the human eye.
        # Through xml_declaration=True '<?xml version='1.0' encoding='utf-8'?>' is printed into the output-file.
        xml_bytes = etree.tostring(jokes, encoding='utf-8', pretty_print=True, xml_declaration=True)
        # xml_bytes.decode('utf-8') decodes the byte-string to 'uft-8'.
        xml_string = xml_bytes.decode('utf-8')

        # The next part opens the output file as 'out' and writes the xml-structure saved in xml_string to the output.
        with open(outfile, 'w', encoding="utf-8") as out:
                out.write(xml_string)

    def save_jokes_json(self, outfile: str) -> None:
        """Save all the jokes of the Generator in their json representation to the outfile"""

        # out_dict is defined in which the elements of each joke will be saved.
        out_dict = {}

        # The for-loop adds a counter to each element of self.jokes through enumerate() and increases the index by +1.
        for i, j_joke in enumerate(self.jokes):
            # The indices will start at 1 and will be the keys of the dictionary, their values are the individual jokes.
            # The jokes are again saved using a JSON representation.
            out_dict[i + 1] = j_joke.json_rep
        # In the 'json_obj'-variable we save out_dict, which is converted to a JSON string by using json.dumps().
        # The parameter 'indent=4' defines the number of indents. It makes a JSON string more easily readable.
        json_obj = json.dumps(out_dict, indent = 4)

        # Lastly, json_obj is written into the defined output-file 'outfile'.
        with open(outfile, "w", encoding="utf-8") as out:
            out.write(json_obj)


if __name__ == "__main__":
    # You can use the following commands for testing your implementation
    gen = JokeGenerator("reddit_dadjokes.csv")
    gen.save_jokes_xml('reddit_dadjokes.xml')
    gen.save_jokes_json('reddit_dadjokes.json')

    gen_json = gen = JokeGenerator("reddit_dadjokes.json")
    gen_json.random_joke()
```

## Encodings
 As we know, not every file you encounter will be encoded in the same way. So, in this task two files will be converted
into *utf-8*.
You recieve two files, encoding_1.txt and encoding_2.txt. One is encoded in ISO 8859-1, the other in ASCII.
For this exercise, I had to:

- find out which file is in which encoding
- convert both files to utf-8 and save them to the same file called encoding_utf-8.txt

```python
def determine_enc(file):
    """This function contains a list of encodings which a text file could possibly have (for this task only 2 are
    possible). The function tries to open the file with each encoding. The right encoding is returned at the end."""

    encodings = ['ASCII', 'ISO 8859-1']

    for enc in encodings:
        try:
            input_file = open(file, 'r', encoding=enc)
            input_file.readlines()
            return enc
        # If a UnicodeDecodeError occurs, the code iterates with the next available encoding in the list.
        except UnicodeDecodeError:
            continue


# The encoding, corresponding to each text file, is saved to the following the variables:
first_enc = determine_enc('encoding_1.txt')  # 'ASCII'
second_enc = determine_enc('encoding_2.txt')  # 'ISO 8859-1'

# The next part opens both text files with their corresponding encoding, determined by the determine_enc-function.
with open('encoding_1.txt', 'r', encoding=first_enc) as file1, open('encoding_2.txt', 'r', encoding=second_enc) as file2:
    # The files are read and assigned to the variables 'text1' and 'text2'.
    text1 = file1.read()
    text2 = file2.read()

    # The encode()-method gives an encoded version of a given string. In this case the strings are encoded as 'utf-8'.
    text1 = text1.encode(encoding='utf-8')
    text2 = text2.encode(encoding='utf-8')

    # In the variable 'out' the variables 'text1' and 'text2' are combined, which is then saved into an output file.
    out = text1 + text2  # b"Why...."

# Lastly, a third file is opened, named 'encoding_utf-8.txt', with the encoding 'utf-8'.
with open('encoding_utf-8.txt', 'w', encoding='utf-8') as new_file:
    # Before the content of 'out' is saved into the file, 'out' is decoded to 'uft-8': bytes are turned to strings.
    new_file.write(out.decode('utf-8'))
```

## Recursive Algorithms

#### The Levensthein-Distance
The Levensthein-Distance
: The Levensthein-Distance, or minimum edit distance, describes the minimum number of changes (deletions, insertions, substitutions) that are needed to convert one string to another.

Example: The Levensthein-Distance of house and home is 2.
1. hose (delete 'u')
2. home (substitute 's' with 'm

```python
def lev(a, b):
  # TODO: Implement the algorithm
    if len(b) == 0:
        return len(a)

    elif len(a) == 0:
        return len(b)

    elif a[0] == b[0]:
        return lev(a[1:], b[1:])

    else:
        return 1 + min(lev(a[1:], b), lev(a, b[1:]), lev(a[1:], b[1:]))


print(lev("house", "home")) # The output is 2.
```

#### Dynamic Programming: Fibonacci Numbers
Recursive algorithms can be simple and powerful, but many of them have the same flaw: They can be **extremly inefficient.**
Since the function ends up calling itself several times in each run, the number of executions stack up fast. The main
problem here is that since the function has no memory, it ends up calculating the same thing over and over again.
This is where dynamic programming comes in: Since the algorithm has no memory, it has to re-calculate the same steps
again and again in every recursion. To solve this issue, we simply add a way to store values we have already calculated:

```python
def dynamic_fibonacci(n):
  # TODO: Implement the fibonacci-algorithm dynamically (with memory storage)
    storage = []

    for i in range(n):
        if i == 0 or i == 1:
            storage.append(i)

        else:
            # Each number is the sum of the two previous numbers- with this in mind, the sequence is generated in the list as follows:
            storage.append(storage[-2] + storage[-1])

    # Lastly, the sum of the last and second last number is returned.
    return storage[-2] + storage[-1]


print("The 10th Fibonacci number:", dynamic_fibonacci(10)) # The answer is 55.
print("The 50th Fibonacci number:", dynamic_fibonacci(50)) # The answer is 12,586,269,025.
```


