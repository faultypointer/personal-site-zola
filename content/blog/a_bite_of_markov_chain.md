+++
title = "Markov Chain"
date = 2023-01-12
draft = false

[taxonomies]
categories = ["A bite of"]
tags = ["A bite of", "python", "ai", "markov chain"]

[extra]
lang = "en"
toc = true
show_comment = true
math = false
mermaid = false
cc_license = true
outdate_warn = true
outdate_warn_days = 120
+++


## What is a Markov Chain?
A Markov Chain is a mathematical model that describes a sequence of possible events, where the probability of each event is only dependent on the previous event.

<!-- more -->

## Example
Suppose there are two states are A and B. At time 0, the system starts in state A. At each time step, there is a probability of 0.7 that the system will stay in state A, and a probability of 0.3 that it will transition to state B. If the system is in state B, there is a probability of 0.4 that it will stay in state B, and a probability of 0.6 that it will transition back to state A.

Here, the state transition diagram looks like this

<img src="/res/blogs/markov_eg.svg" height=350, width=350>

So, you can see that the probabilities of transitioning from one state to another are fixed, and depend only on the current state.


## Ai-minem: My attempt at Markov Chain

It is the simplest form of markov chain. What I did was take a bunch of lyrics of eminem's song and use it to train a model then had the model spit out a rap. Lets go through the project:

### Collecting Data

I used requests to get the html data of the lyrics page and beautifulsoup4 to parse the data.

```python
lyric = ""
res = requests.get(url)
page = BeautifulSoup(res.content, "html.parser")
for div in page.find_all(attrs={"class": "Lyrics__Container-sc-1ynbvzw-6"}):
    for link in div.find_all("a"):
        lyric += link.text.strip()
        lyric += "\n"
```


### Markov Model
Markov model is a class with members data and chain. Data is used to store the training data and chain is used to store the array of words that come after the current word.

```python
class Markov:
    def __init__(self) -> None:
    self.data = []
    self.chain = {}

    def add_data(self, data) -> None:
        ...
    def train(self):
        ...
    def spit_bars(self, n=100):
        ...
        
```


### Training the Markov Model
The data I collected are array of strings where single string is the whole lyrics of a song. For training the model, I looped through each string(lyrics) and used regex to find all the words in that lyrics. Then for each word, I saved the words that come after it in the *chain* member. 

```python
def train(self):
    for text in self.data:
        words = re.findall(r'\w+', text)
        for (i, word) in enumerate(words[:len(words)-1]):
            if word in self.chain:
                self.chain[word].append(words[i+1])
            else:
                self.chain[word] = []
```


### Spitting Bars
Using the model to produce the lyrics is done through spit_bars function. It takes one optional argument, n, which denotes the number of words to produce. First, it selects a random word from the dictionary and saves it in the current variable. Then it loops n times, each time printing the current variable. Then it gets the list of possible choices using the current word as key from the chain dictionary. If the possible choices array is an empty, then all the keys from the dictionary become the possible choices. Then it sets the value of current to random word from possible choices and repeat.

```python
def spit_bars(self, n=100):
    default_choices = list(self.chain.keys())
    current = random.choice(default_choices)
    i = 1
    while i <= n:
        if i % 15 == 0:
            print("")
        print(current, end=" ")
        try:
            choices = self.chain[current]
        except KeyError:
            choices = default_choices
        
        if choices:
            current = random.choice(choices)
        else:
            current = random.choice(default_choices)            
        i+=1
    print("")
```


### Output
The markov model produced this output after training with 5 eminem's songs lyrics.

	suffer Kneel autumn Spider throatin above missed This flippity type of money capI Made 
	bigger help your concert heart readyTo cheese dope Bill truthful throat spray and shoot ya 
	bombs fact this hole momentWould RakimLakim killed attack Fat are so bad it s a 
	Doberman us to choke move like to a lifetime yoYou better never let it was 
	a catchy heavyThere old in commonWe are hungry I got them uppers looks superstardom knees 
	flippity reason is drop bombs explosions the trailer s all in the blueprint ma sleep 
	and on the trunkBut 2004 Think gone cold I put em with your partners feet 
	Did idolHe while he won t see at all And here s not miss your 
	ass disastrously killers Make maxi pad Evil cut your mind to set your headSo talk 
	rubber slip Pharoahe paraplegic the front to fuse audience totterCaught toward honestBut jealous While an 
	MC realized fall positionTo May back pocket tongues These hoes is gaping f inadvertently arsenal 
	chips forgot really got You don t know how much we re sayin face lookin 
	boy cause I ve got some And even been up you scream which six years 
	finna kill himself carry figure outHow to tell me to the fuck that anything you 
	already know the Air Breezy lead Apache add jointsProlly burst at the fuck it s 
	how much we have arms are over some bitch I could it life for sure 
	you re cut your posters use sublim problem lose yourself like to feel like Apache 
	Collins together think I m Back cold product of me know there to bite aStrap 
	Up on Cause I m a product They moved Glock capableOf openMy stacked rarely Syllables 
	read phat soldierTongue Get outta those days t utilize murderers or shot do thoughFor way 
	to come at me do not miss your Gang Like you better lose yourself in 
	a couple of RakimLakim jot reminds me like a pile 9See dawg room yackety ago 
	helps voices vodka you better lose yourself like his tough demeanourSo songsSo buggin tGive caught 
	lemons dah dah dah dah You re never asked change It s my family IGot 
	out Chorus Dido My tea s how much it only grows hotter if I m 
	beginnin to the flowsAnd B Up on her throat spray and Audemars jointsProlly Mom capture 
	this is such swallow huh This little brother I can t breathe genital comin at 
	me outside cornrows spot myself cause I ain t utilize Jordans ma name across coaster 
	lead Syllables was a costly beginning along truthful finna kill you can t batter GoldmanI 
	Slim I did that made itSo insult matter ColumbinePut fatal N9neIf wondering why IGot out 
	like a time Shady you tell me notThis flippity sendin answer all hell yeah from 
	your nose dove venom pull Bizarre after Mya said on my life And what I 
	can barely knows his name was youDamn amplified skunk talk calm trunk white gotta dance 
	long as fuck that s how 

## What's next?
I plan to make it so that instead of using just one word as a determining factor for the next word, it uses maybe 3 or 4 previous words to do so. I also want to use neural network or ML algorithm. Although I will probably forget about this since I started working on another project.


That's it for today, see you in next blog.