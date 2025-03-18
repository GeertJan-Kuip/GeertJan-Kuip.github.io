## Functional programming

As there is a lot to learn about streams and functional programming, and as it is hard to learn it purely from a book, I decided to see for myself whether I could make something interesting with it. 

Below is code which I used for the following:
- from gutenberg.org I downloaded text files of three books, namely Moby Dick, Oliver Twist and Pride and Prejudice.
- I put these in the project folder and created a simple class Book (I did not include it in the code below)
- A book object contains a field of type Path, pointing to the text file
- Starting with the three paths, the code tries to find the ten most frequent used words written with a capital, excluding "I"
- If it works well enough, we should see the main characters appear in the end result

The code:

```
Stream.of(book1, book2, book3) --I created a class Book with a Path field. 
        .collect(Collectors.toMap(Function.identity(),book->{
            String text = null;
            try {
                text = Files.readString(book.getPath());
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
            return text;
        }))

        .entrySet().stream()
        .map(entry->{
            String[] list = entry.getValue()
                    .replaceAll("[\\r\\n]"," ")
                    .replaceAll("\\s+", " ")
                    .replaceAll("Mr.", "Mr")
                    .replaceAll("Mrs.", "Mrs")
                    .replaceAll("\\. [A-Z]", "\\. ")
                    .replaceAll("  [A-Z]", " ")
                    .replaceAll("[A-Z][A-Z]", "[a-z][a-z]")
                    .split("[\\.?!,\\s\\s+]");
            List<String> list2 = new ArrayList<>();
            Map<String, Integer> frequencyTable = new HashMap<>();
            for (String word : list)
                if (word!="" && word.length()!=1 && word.charAt(0)>64 && word.charAt(0)<91)
                    frequencyTable.merge(word,1, Integer::sum);

            List<Map.Entry<String,Integer>> orderedFrequencyList = new ArrayList<>(frequencyTable.entrySet());
            Collections.sort(orderedFrequencyList, new Comparator<Map.Entry<String,Integer>>(){
                @Override
                public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
                    return o2.getValue()-o1.getValue();
                }
            });
            
            return Map.entry(entry.getKey(), orderedFrequencyList.subList(0,20));
        })
        .forEach(x->System.out.printf("%s - %s\n\n",x.getKey().getPath().getFileName(),x.getValue()));
```

The result:

```
Moby Dick.txt - [Ahab=379, Whale=210, Stubb=200, Queequeg=184, Captain=178, Starbuck=146, Sperm=133, Pequod=125, God=92, The=91]

Pride and Prejudice.txt - [Mr=954, Elizabeth=471, Darcy=359, Miss=263, Jane=231, Bingley=222, Bennet=153, Wickham=141, Collins=135, Lydia=114]

Oliver Twist.txt - [Mr=1189, Oliver=620, Bumble=320, Sikes=288, Jew=271, Fagin=261, The=167, Brownlow=144, Rose=139, Monks=117]
```

First evaluation:
- It sort of works. Ahab, Whale, Elizabeth, Mr Darcy, Oliver, Bumble, they are on top with frequencies.
- I'm terrible with regex, needed to google and use ai assistent a lot.
- The try-catch, mandatory because of ```Files.readString``` takes a lot of lines.
- Furthermore the code is reasonably compact. I like that.
- I think for others it will take quite some time to understand how the code exactly works.

What I learned
- While the code gets less verbose using functional programming/streams, creating it still takes much time. There is effort in every detail.
- I think there are too many methods and classes available to become fluent in this part of the language, unless you specialize.
- Fortunately there is good documentation (docs.oracle.com).
- Java has really great methods. Files.readString() and the Collectors class methods for example.
- The .merge() method from the Map interface is amazing. It puts a new entry in the map, or not if it is already there, and if so, you use the BiFunction to set a new value.
- I learned to play with Map.Entry, it is in my vocabulaire now.
- I wrote a Comparator to be used with Collections.sort.
- One day I will learn better regex.

All in all a rather rewarding experience, really learned a lot.
 
