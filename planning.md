# Planning: r/gamedesign Post Classifier

## Community

I chose **r/gamedesign**, a subreddit focused specifically on game systems, mechanics, player experience, and design theory. Its narrower design focus makes it more suitable for this classification task than broader game-development communities, which also contain programming, career, marketing, and general production discussions.

This community is a good fit for classification because its posts serve several genuinely different purposes. Some posts debate broad principles that could apply across many games. Others ask for help creating or balancing a particular gameplay system. Some center on how players think, feel, learn, or behave while playing. Others discuss the process of researching, prototyping, testing, documenting, or changing a game project.

A regular participant in the community would recognize these as different kinds of discussions. A post asking, "How do I balance my skill tree?" serves a different purpose from one arguing that games should reward curiosity over optimization. This classifier is intended to learn those distinctions.

## Labels

### 1. `design_theory`

A post that explains, questions, or debates a broad and reusable principle of game design. It is generally framed across multiple games, genres, or possible implementations rather than being limited to one system the poster is currently building.

A post can mention a specific game as an example and still belong to `design_theory` when the example supports a broader, generalizable argument.

**Example 1:**

> Why 3 Options? Numerology in games.
>
> Want to know why so many games offer just 3 choices at a time? Slay the Spire with card picks, Hades with its boons, and many more default to just 3 options to pick from. Why exactly is that?
>
> Here's why: It's baked into our brain chemistry. There's also specific reasons to use 2, 4, 5, or 8. Let's talk about sets of 3 first.
>
> Our brains don't usually judge the value of each option independently the way economists would prefer, we tend to judge by comparison. Extensive studies show that we irrationally like an option better if you offer us an extra option that is obviously worse. [...]

This is `design_theory` because the post argues a reusable principle (the decoy effect in choice architecture) that applies across many games — Slay the Spire and Hades are cited as supporting evidence, not as the subject the post is actually about.

**Example 2:**

> Why does optimal play so often kill the fun, and how do we fix that at the design level?
>
> There's a tension I keep thinking about in game design between optimal play and interesting play. In a lot of welldesigned games, the most effective strategy is also kind of the dullest one. Players who figure out the dominant approach just repeat it, and the game becomes a checklist rather than a series of meaningful decisions.
>
> Halo came up recently as an example of a game that nudges you toward using grenades naturally, without forcing it. [...] So the question I'm chewing on: how do you design systems that make the interesting choice and the effective choice overlap more often? [...] Curious whether anyone has examples of games that genuinely nail this, or design frameworks that address the gap between optimal and interesting. Board games and tabletop RPGs welcome too, not just video games.

Note that Halo also appears in a `player_experience` example below, for a different post asking why the poster personally underutilizes grenades — see the removal test under Hard Edge Cases for how the same game ends up under two different labels.

### 2. `mechanic_design`

A post focused on creating, balancing, implementing, or evaluating a particular gameplay system, rule, interface, encounter, progression structure, or interaction.

A post may discuss the desired player response, such as satisfaction or frustration, but it is labeled `mechanic_design` when the main question is which concrete rules or systems should produce that response.

**Example 1:**

> Picking a resource type for a 4X
>
> Hello, I'm currently working on a PvE 4X game where the player expand on an hostile content, increasingly reaching out to gather more materials.
>
> One of the design parameters I've been unable to settle with is the way to handle strategic resources. Roughly speaking there could be two choices: treating them as stockpiles, slowly filling up, or only as available flux, without accumulation. [...] I'm tempted to try the flux version for everything, but I'm afraid players could find this very limiting if they hit unit caps too fast. I think both system work but I'd like to hear about your experience with games implementing them.

This is `mechanic_design` because the question is entirely about a specific resource-handling decision for the poster's own 4X game, with no claim being made that's meant to generalize beyond it.

**Example 2:**

> Should classes be balanced to similar win rates, or is intentional difficulty variation a feature?
>
> I'm building a dungeon solitaire game with multiple playable classes, and the win rates across them are all over the place. Some sit in the high 80s, while the weakest is down at 34%.
>
> My first instinct was to "fix" this: nerf the easy classes, buff the hard ones, and pull everything toward a tighter band. But the more I sit with it, the more I wonder if that's the wrong goal.
>
> What if the spread is the feature? A new player or someone who just wants a relaxing run picks an easy class, and someone chasing a real challenge picks the 34% one. [...]

(This one is discussed as a difficult labeling decision under Baseline Results below — the generalizable-sounding framing is a trap.)

### 3. `player_experience`

A post centered on how players think, feel, perceive, choose, learn, identify, or react while playing.

This label is used when player behavior, emotion, motivation, perception, or preference is the main object being examined. It is not used merely because a mechanic is intended to affect players; nearly every design post could meet that weaker condition.

**Example 1:**

> When players are given the choice to be good or evil, they always choose to be good. Are there any games that manage to prevent this?
>
> The title might be a bit of an exaggeration, but I've been gaming for about 20 years and have been watching others play for a long time. What I've noticed is that if given a choice, almost everyone chooses to be a good character, at least during their first playthrough. In fact, a BioWare producer mentioned on Twitter that 92% of Mass Effect players chose to be Paragon [...] Is there a game where the player struggles to choose [...]? The only exception that comes to my mind is Frostpunk. [...] Are there any other games like this?

This is `player_experience` because the post's central question is about an observed pattern in player behavior. Removing the supporting evidence (the Mass Effect statistic, the Molyneux quote) still leaves a complete behavioral question; this distinguishes it from `design_theory`, where a behavioral example would instead be evidence for a structural claim about morality systems.

**Example 2:**

> What is it with Halo that makes me not underutilize grenades?
>
> or rather, what is it with other FPS/TPS games that makes me underutilize grenades?
>
> no matter what I never stocked up on grenades in Halo. You kill something, it drops one I immediately throw one. And yet Halo isn't that sophisticated when it comes to grenades [...] I'm curious as to why tho? is it because in Halo its Q and not G like in other games? destructible cover [...] fast throwing animation? or maybe is it just me all along?

The same game (Halo) appears under `design_theory` above, for a different post arguing a transferable principle about optimal vs. interesting play — these two examples together demonstrate the removal test in the Hard Edge Cases section below.

### 4. `dev_process`

A post about the process of making, researching, planning, testing, documenting, presenting, or revising a game project rather than primarily about the game's content. This label also includes requests for books, papers, tutorials, research methods, and practical guidance on learning game design.

**Example 1:**

> Mina the Hollower had an 800+ Page Design Doc
>
> Some colleagues and I were recently in a call with Alec Faulkner, a game designer at Yacht Club Games, playing through the opening of Mina the Hollower and talking about its design. When someone in the chat asked about what Mina's design documentation looked like, he showed us their 800+ page design document. [...] Alec made it clear that the paper and whiteboard design process IS the main design process for them [...] Everything was exhaustively worked out from the start, and when things changed they updated the documentation. [...] So I wanted to ask you all, what kind of games do you work on and how do you approach documenting their design?

This is `dev_process` because the post is about a studio's documentation and pre-production practices — the subject is *how* the game gets made, not any specific mechanic in Mina the Hollower itself.

**Example 2:**

> I wanted my game to have procedural generation but does it really need it?
>
> I am making a simple game where you go from room to room solving puzzles. At first I made some procedurally generated rooms [...] For the most part it seemed like it was going well until I started to think of what the game actually was. [...] So in the long run I just abandoned proc gen and built out the levels how I think it would go. I might revisit it, but my question is: I wanted my game to have procedural generation but does it really need it?

(The `mechanic_design` versus `dev_process` boundary section below explains why this counts as process rather than system design.)

## Hard Edge Cases

### `design_theory` versus `player_experience`

The most common ambiguous case is a post that uses player behavior as evidence for a broad design principle.

If the main purpose is to establish a reusable claim about game design, the post is labeled `design_theory`. If the main purpose is to understand how players perceive, feel, learn, choose, or behave, it is labeled `player_experience`.

A useful test is to remove the supporting material:

- If the broad design argument remains complete without the behavioral example, the post is probably `design_theory`.
- If the behavioral observation remains the central question, the post is probably `player_experience`.

For example, a post asking why players usually select morally good options is primarily about behavior and belongs to `player_experience`. A post using that behavior to argue that binary morality systems are structurally ineffective may instead belong to `design_theory`.

### `mechanic_design` versus `player_experience`

Many posts describe a desired feeling while asking how to design a system. For example, a post may ask which placement previews, snapping rules, sounds, or visual cues make first-person construction feel satisfying.

The desired feeling does not automatically make the post `player_experience`. If the post is mainly asking which concrete controls, rules, feedback systems, or interface elements should be implemented, it is labeled `mechanic_design`.

The post is labeled `player_experience` when perception or emotion itself is the main subject, such as why large objects often fail to feel imposing or why repetition sometimes feels relaxing and sometimes causes burnout.

### `mechanic_design` versus `dev_process`

Posts about a designer's own project often include both a system and the process through which it was created.

A post is labeled `mechanic_design` when the final system itself is the main subject. It is labeled `dev_process` when the central subject is researching, prototyping, playtesting, documenting, scoping, revising, or deciding whether to pivot.

A postmortem is not automatically `dev_process`. For example, a post describing playtesting that led to a new reward system may still be `mechanic_design` if the post mainly asks whether the new reward rules work. It becomes `dev_process` when the sequence of experimentation, failure, feedback, and redesign is the point of the post.

This applies to the `dev_process` procedural-generation example above: the post is about reconsidering a production decision (trying proc-gen, recognizing it didn't fit the project, abandoning it) rather than about a specific mechanic. If the same poster had instead asked "how do I guarantee the proc-gen system never locks a required ability behind an unsolvable room," the system itself would be the subject and the post would be `mechanic_design`.

### Resource Requests

Resource requests are classified according to their communicative purpose.

Requests for books, academic papers, tutorials, learning strategies, or research methods are labeled `dev_process`, even if the requested subject concerns mechanics or player behavior. The post is fundamentally about how the designer will learn, research, or prepare.

Requests for examples of a particular mechanic may instead be labeled `mechanic_design` or `design_theory`, depending on whether the question is about a specific implementation or a general design pattern.

For example:

- "What academic papers exist on combat-oriented playstyles?" is `dev_process` because it is a literature request for a thesis.
- "Are type advantages necessary in turn-based games?" is `design_theory` because it asks about the general role of a design pattern.
- "What games use GOAP for non-soldier NPCs?" is `mechanic_design` because it asks about implementations of a particular AI system.

### Opinion Polls

Opinion polls are classified according to what respondents are being asked to evaluate.

- Questions about what players enjoy, prefer, dislike, or find frustrating are `player_experience`.
- Questions asking whether a specific proposed system sounds effective are `mechanic_design`.
- Broad questions about how a genre or design pattern should evolve are `design_theory`.

For example, "What game mechanics do you find most fun and why?" is `player_experience`, while "Would a gridless suitcase inventory provide too much freedom?" is `mechanic_design`.

### Posts Without a Valid Fit

I considered using a fifth `skip` or `other` label but excluded it to keep the taxonomy compact and because the assignment requires the labels to cover at least 90% of the selected posts.

Clearly promotional posts, moderator announcements, unrelated programming questions, advertisements, and posts that are too vague to classify reliably are excluded during data collection instead of being forced into an inaccurate category.

## Data Collection Plan

The final dataset contains **200 posts from r/gamedesign**. Posts were selected from a larger export of subreddit content and manually reviewed for relevance. Advertisements, promoted posts, moderator threads, unrelated discussions, and entries without enough design-related substance were excluded.

The text used for classification preserves the post title and the relevant body. Usernames, vote counts, advertisements, navigation text, and other Reddit metadata were removed because they should not influence the classifier.

Duplicate detection was performed in multiple stages:

1. Repeated titles and bodies were checked manually.
2. Text was normalized by removing differences in capitalization, punctuation, and extra whitespace.
3. Exact duplicates were removed.
4. A text-similarity check was used to identify lightly reworded or near-duplicate examples.
5. Conceptually similar posts were reviewed to distinguish genuine duplicates from separate posts discussing the same design topic.

One duplicate concerning the perception of large-scale objects was removed and replaced. The final dataset contains no normalized exact duplicates and no near-duplicate pairs above the similarity threshold used during checking.

The final class distribution is:

| Label | Examples | Percentage |
|---|---:|---:|
| `mechanic_design` | 76 | 38.0% |
| `design_theory` | 61 | 30.5% |
| `dev_process` | 38 | 19.0% |
| `player_experience` | 25 | 12.5% |
| **Total** | **200** | **100%** |

The classes are not perfectly balanced, but the distribution reflects the fact that concrete mechanic questions are especially common in r/gamedesign. The dataset was rebalanced by replacing several weak or overly narrow `mechanic_design` examples with stronger `dev_process` and `player_experience` examples from the larger subreddit collection.

I did not force every category to contain exactly 50 examples because doing so would have required accepting weaker or less representative posts merely to achieve numerical equality. During model training, class weighting or another imbalance-aware method may be used to reduce bias toward the larger classes.

## Dataset Splitting

The dataset will be divided into training and held-out testing sets using stratified sampling so that all four labels are represented.

The test set will contain at least 10 examples from each class, for a minimum of 40 test examples. This is important because per-class precision and recall based on only a few examples would be too unstable to interpret.

Posts that are close conceptual variants of one another will be reviewed before splitting. When multiple posts address nearly the same question, they should be assigned to the same split where practical. This reduces the risk of topic leakage, in which the classifier sees an extremely similar example during training and receives an artificially high test score.

The final test data will not be used to revise the labels, tune preprocessing decisions, or select the best model. If model comparison or parameter tuning is required, a validation split or cross-validation procedure will be used on the training data.

**Note on actual split achieved:** with 200 total examples and a 70/15/15 stratified split, the actual test set landed at 30 examples (9 `design_theory`, 12 `mechanic_design`, 4 `player_experience`, 5 `dev_process`) rather than the 40+ originally targeted above. `player_experience` in particular has only 4 test examples, below the 10-per-class minimum I originally specified as the threshold for stable per-class metrics. I'm flagging this explicitly: per-class recall/precision for `player_experience` and `dev_process` should be read as directionally suggestive rather than statistically robust, since a single misclassification shifts recall by 20-25 points on these classes.

## Evaluation Metrics

Accuracy alone is not sufficient because the labels are not evenly distributed. A classifier that predicts `mechanic_design` too frequently could achieve a seemingly reasonable accuracy while performing poorly on `player_experience` and `dev_process`.

The following metrics will be used:

- **Per-class precision:** Measures how often predictions of a particular label are correct.
- **Per-class recall:** Measures how many true examples of each label the classifier successfully identifies.
- **Per-class F1:** Combines precision and recall for each category.
- **Macro-averaged F1:** Calculates F1 independently for every label and gives each class equal weight.
- **Confusion matrix:** Shows which label pairs the classifier most frequently confuses.
- **Overall accuracy:** Included as a secondary summary metric, but not used alone to judge success.

Macro-averaged F1 is especially important because it prevents the larger `mechanic_design` and `design_theory` categories from dominating the evaluation.

I expect many errors to occur on the following boundaries:

- `design_theory` versus `player_experience`
- `mechanic_design` versus `dev_process`
- `mechanic_design` versus `player_experience`

The confusion matrix will help determine whether the classifier is struggling with these expected ambiguities or failing to learn the broader taxonomy.

## AI Tool Plan

AI tools were used as annotation and consistency aids during the creation of the dataset. They were not treated as an unquestionable source of ground-truth labels.

### Taxonomy Development and Stress-Testing

An AI assistant was given proposed label definitions and asked to apply them to many real r/gamedesign posts. Difficult examples were discussed individually to determine why one label was stronger than another.

This process helped identify ambiguous boundaries, including:

- Player experience versus the mechanics intended to create that experience
- General design theory versus project-specific system design
- Mechanic design versus development process
- How to classify resource requests and broad opinion polls

The written edge-case rules in this plan were developed partly from those disagreements and discussions.

### Annotation Assistance

AI assistance was used to suggest labels, difficulty levels, and explanations for examples. Each classification was reviewed in the context of the full title and body rather than accepted solely from the model's first response.

The AI-generated explanations were used to check whether the chosen label could be justified using the written taxonomy. When a suggested classification conflicted with an earlier decision or with the label definitions, the distinction was discussed and refined.

The final dataset therefore represents AI-assisted human annotation rather than fully automatic labeling. The labels remain the responsibility of the project author.

### Duplicate and Balance Checking

AI and programmatic tools were used to:

- Identify possible duplicate or near-duplicate posts
- Review class counts
- Locate underrepresented categories
- Suggest stronger replacement examples from the larger subreddit export
- Check whether replacement examples were already present

These tools supported dataset cleaning, but possible duplicates and replacements were also reviewed manually.

### Failure Analysis

After evaluation, I will provide the AI tool with a structured list of misclassified examples containing the true label, predicted label, and relevant text.

I will ask it to suggest possible patterns, such as:

- Errors clustering around a particular label boundary
- Resource requests being disproportionately misclassified
- Short or vague posts causing more errors than detailed posts
- Vocabulary such as "players," "feel," or "prototype" misleading the classifier
- Project-specific posts being mistaken for general theory

I will manually verify each proposed pattern against the actual examples before including it in the final analysis. AI-generated summaries can overstate patterns based on only a few cases, so they will be treated as hypotheses rather than conclusions.

### Limits on AI Use

AI will not be used to replace final evaluation or to conceal uncertainty in the annotations. Ambiguous examples will remain documented through the dataset's notes rather than being presented as objectively indisputable labels.

The final report will disclose that AI assistance was used during taxonomy refinement, annotation discussion, duplicate checking, dataset balancing, and failure analysis.

## Baseline Results (Milestone 4)

I ran the zero-shot Groq baseline (`llama-3.3-70b-versatile`) on the locked 30-example test set, using a classification prompt built from the label definitions and edge-case decision rules above (full taxonomy text, one worked example per label, and explicit removal-test rules for each boundary described in Hard Edge Cases).

**Overall accuracy: 0.83** (25/30 correct; zero unparseable responses out of 30).

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `design_theory` | 0.70 | 0.78 | 0.74 | 9 |
| `mechanic_design` | 0.92 | 0.92 | 0.92 | 12 |
| `player_experience` | 1.00 | 0.75 | 0.86 | 4 |
| `dev_process` | 0.80 | 0.80 | 0.80 | 5 |
| **Macro avg** | **0.85** | **0.81** | **0.83** | 30 |

This baseline already clears every quantitative target in the Definition of Success section below, before any fine-tuning occurred:

- Macro F1 (0.83) clears the 0.70 target.
- `player_experience` recall (0.75) and `dev_process` recall (0.80) both clear the 0.70 target.
- The lowest per-class recall observed (0.75) is well above the 0.50 floor.

The weakest single number is `design_theory` precision (0.70) — some non-`design_theory` examples are being predicted as `design_theory`, consistent with my own prediction that the `design_theory` ↔ `player_experience` boundary would be the hardest one in the taxonomy. Given the small support sizes here, especially `player_experience` (n=4), I'm treating these per-class numbers as directionally informative rather than statistically precise — one flipped prediction shifts recall by 20-25 points on the smaller classes.

This result reframes what fine-tuning needs to demonstrate. Rather than simply asking whether fine-tuning produces a usable classifier, the more honest question given this baseline is whether a small, label-naive model (DistilBERT, with no pretrained notion of these four categories) can match a 70B-parameter model that received the entire taxonomy, worked examples, and edge-case decision rules as natural-language instructions. My hypothesis entering fine-tuning is that it will **not** match the baseline: 140 training examples is a small amount of data for a model with no prior conceptual grounding to learn four genuinely overlapping categories from raw text alone, and a meaningful share of the baseline's performance plausibly comes from the explicit edge-case rules in the prompt rather than from the labels themselves — rules a fine-tuned classifier has no equivalent access to.

## Definition of Success

The classifier will be evaluated on a held-out test set. The originally targeted minimum of 10 examples per class (40+ total) was not achieved at the actual 200-example dataset size and 70/15/15 split — see the note in Dataset Splitting above.

The main success targets are:

- **Overall macro-averaged F1 of at least 0.70**
- **Recall of at least 0.70 for `player_experience`**
- **Recall of at least 0.70 for `dev_process`**
- **No individual label with recall below 0.50**

These requirements are intended to prevent a model from appearing successful by predicting the two larger classes while largely ignoring the smaller ones.

The confusion matrix will also be examined to understand the nature of the remaining errors. I expect a portion of the mistakes to occur between conceptually adjacent labels, particularly `design_theory` and `player_experience`, or `mechanic_design` and `dev_process`. However, this is a diagnostic expectation rather than a required percentage of errors.

A small number of errors spread across several label pairs would not automatically make a strong classifier unsuccessful. Conversely, a large number of errors concentrated on one expected boundary would not be acceptable simply because the boundary was predicted to be difficult.

I would consider the classifier suitable for this project if it meets all quantitative targets and manual review shows that most remaining mistakes involve genuinely ambiguous posts rather than obvious failures to understand the categories.

For deployment in a real community tool, I would additionally require:

- Manual review of low-confidence predictions
- Periodic checks for shifts in subreddit content
- Monitoring of class distribution over time
- A process for revising the taxonomy when recurring new post types no longer fit the existing labels