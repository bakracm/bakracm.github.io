# Mina's Competencies

## Development

**Medium-scale Abstraction**

Sam and I co-wrote the three UI bet components which employ medium-scale abstraction. This consists of a bet feed (sam wrote and I edited), a sidebar with top bets (I wrote), and a minimization option for a bet feed that displays less information than the regular
feed post. The code sample abstracts away repetitive behavior such as connecting the props for each piece separately. Instead, the same information is passed into all of them to ensure that they update in real time.


For example, If I were to update something in a post on the bet feed, such as add a wager, it would automatically update the value on the top bets sidebar. Below is the my code for topBets:

```
export default function TopBets({ posts = [] }: TopBetsProps) {
  const topPosts = (Array.isArray(posts) ? posts : [])
    .filter(
      (p) =>
        typeof p.leftTotal === "number" &&
        typeof p.rightTotal === "number"
    )
    .slice()
    .sort(
      (a, b) =>
        (b.leftTotal + b.rightTotal) -
        (a.leftTotal + a.rightTotal)
    )
    .slice(0, 5);

  return (
    <div style={styles.container}>
      <h2 style={styles.title}>
        Top Bets
      </h2>

      {topPosts.length === 0 ? (
        <div style={{ fontSize: "14px", color: "#555", textAlign: "center" }}>
          No bets yet
        </div>
      ) : (
        topPosts.map((post) => {
          const total = post.leftTotal + post.rightTotal;

          return (
            <div
              key={post.id}
              style={{
                padding: "8px 0",
                borderBottom: "1px solid #eee",
              }}
            >
              <div style={{ fontWeight: "bold" }}>
                {post.title}
              </div>

              <div style={{ fontSize: "13px", color: "#555" }}>
                Pool: {total} acorns
              </div>

              <div style={{ fontSize: "12px", color: "#777" }}>
                {post.leftLabel}: {post.leftTotal} |{" "}
                {post.rightLabel}: {post.rightTotal}
              </div>
            </div>
          );
        })
      )}
    </div>
  );
}e
```
It is called inside the Main App.tsx:

```
  <aside style={cardStyle} aria-label="Top Bets">
    <TopBets posts={posts} />
  </aside>

```
Which is nice because it helps me avoid cluttering while still syncing up data from the bets. I don't have Sam's code for
the additional component, but he also has a separate file for small bet which he makes a call to.

**Medium-scale Architecture**

I was responsible for co-designing the front-end of our project. The pattern here was to use TypeScript + React and have a main
App.tsx file which calls on a bunch of smaller components to create the UI. We made this design choice in an attempt to decompose
and avoid cluttering our App file with things like component-specific types and styles.


The result of this was one main App.tsx, and seven additional TypeScript component files: button, betPost, Currency, Leaderboard,
Login, Registration, and TopBets. The imports for these files were primarily ones from react and imports of other component files.
The exception were the registration and login components for which we used the react-hook-form library to handle the forms when you 
are prompted to sign in or create a new user. Additionally, we created test files that correspond with each of the components, and 
those contained vitest imports.

Most of the components get called in the App.tsx:

```
import banner from "./components/gambling-banner.jpg";
import Button from "./button";
import Post from "./betPost";
import Leaderboard from "./Leaderboard";
import Currency from "./Currency";
import Login from "./Login";
import Register from "./Registration";
import TopBets from "./TopBets";

```


## Verification

**Testing Dimensions**

I was responsible for writing tests for the front-end work that Sam and I developed. I set everything up for the team and then wrote
tests on the Leaderboard and Currency components for other to use as example. I had intended on helping out with back-end testing, 
but Youssef and Lucas had done pretty much all of it by the time our meeting wrapped up. After revising our project, I noticed that
some of the tests that Sam had written were failing, so I fixed those and added additional tests to his files.


The coverage was 100 for most of the component files. Our main app was not because we chose to not test the minigames. 
The tests for the UI are generally check whether the components show up like they are supposed to. For example, whether they pop-ups 
and containers have correct words and values when rendered. Some edge cases here 
checked what would happen if a component only got fed one line of data, or even none. For example, what would our leaderboard display
if there was only one registered user? What would it display for none? These tests were harder to come up with because we had already
reached 100% coverage for those files. Additionally, a few of our components, such as TopBets and Leaderboard, dealt with logic in
the front-end, so for those, we dug deeper than the "does this display correctly?" testing process.


We made the decision to not cover the mini-games with our tests. Despite the mini-games being a UI component, they were in big part
AI slop and a last-minute addition to our web app. I think this is apparent as they don't really fit into the aesthetics of the UI,
and would likely be reworked if we continued work on the project. The verification for this part was manuel, we spent a lot of time
playing our mini-games trying to see if any major issues would arise. We did not catch onto anything big.

Here is one of the test from my leaderboard, it takes in a list of mock user props and makes sure the leaderboard sorts in the correct order:

```
test("sorts users by acorns descending", () => {
    render(<Leaderboard players={mockUsers} />);

    const rows = screen.getAllByText(/acorns/);

    expect(rows[0]).toHaveTextContent("1000 acorns");
    expect(rows[1]).toHaveTextContent("80 acorns");
    expect(rows[2]).toHaveTextContent("80 acorns");
    expect(rows[3]).toHaveTextContent("70 acorns");
  });

```

**Testing Infrastructure**

For testing infrastructure, I used vitest which required running a single command in the terminal (npx vitest) for all tests. 
I took on the responsibility of setting up my team's testing and figuring out how to put everything together. I noted which packages
needed to be installed and then walked my group through it once i got it up and working for an example file. This included me 
updating the appropriate .json files first and then creating a test folder with a setup file that does the imports. The front-end
testing was pretty straight-forward, we had a different test file for each component/file that corresponds to the name.tsx with 
name.test.tsx.

Running "npx vitest --coverage" was pretty helpful, it ran all of our tests at once for both the front and back ends. As a team, we managed to uncover one major bug with testing. Anyone could create a bet without logging in, which we were not intending on having
and we did not realize when we clicked through our program.

This was the test that uncovered the bug:

```
test('createMarket validates input before inserting', async () => {
    const client = createClient();
    mockDb.connect.mockResolvedValue(client);
    client.query.mockResolvedValueOnce().mockResolvedValueOnce();

    await expect(createMarket('', '', 'desc', 'yes', 'no', '2099-01-01T00:00:00.000Z')).rejects.toThrow('Invalid market data');
    expect(client.query).toHaveBeenNthCalledWith(1, 'BEGIN');
    expect(client.query).toHaveBeenNthCalledWith(2, 'ROLLBACK');
  });
```

## Design

**Needfinding**

For our needfinding process, we thought it would be best to interview our friends and peers (seeing as our target audience was
Grinnellians). We came up with a list of base questions to ask in order to assess interest for our app, and we asked for opinions
on specific features we were considering to evaluate needs. We were very intentional with our interview subjects, making sure they
differed in interest (frequent gambler, non-gambler, ex-gambler, etc.). Some of our questions related to other social media 
platforms that we knew our peers used. We learned, for example, which features Grinnellians prefer on the forum app "YikYak" as 
well as which ones they do not care for, which helped narrow down our list of potential features to implement.


Some of our most valuable takeaways were that Grinnellians longed for a platform where they could build campus community. The
coolest thing about yikyak is not the anonimity, but the relatability of a personalized feed. Additionally, we gathered that our 
potential users would enjoy gambling more it involved a fake currency. This way, there was less risk associated with the 
entertainment. 

**Ideation**

For ideation, my team and I created personas to help us brainstorm ideas. We tried to be very specific with the descriptions of each
characters, because it left room to create more characters to fill gaps. We thought through what each of them would want and why they
were using our app. Then we used this as a point of reflection for how to optimize the UI to fit their needs. For example, someone who
is using the app socially would want to show off their success, so a good feature would be a leaderboard. We had another persona that
would lurk on the app to stay in the loop on what's going on on campus. Since all they did on the app was scroll, we decided that
anyone can view bets without being prompted to create a user.


This approach showed to be very helpful as most of our components were thought of during that session. We did not develop all of them,
as we ran out of time and had not gotten to stuff like tabs for categories. Regardless of this, it was one of the more helpfful things we did

<img width="3024" height="4032" alt="593131980-59ceb58d-c41c-43c3-b0ac-da196af86bf6" src="https://github.com/user-attachments/assets/9660002c-30f7-4aba-b341-71792296373e" />
<img width="3024" height="4032" alt="593131993-bde02a8b-d672-4f05-9612-259d71716d35" src="https://github.com/user-attachments/assets/0f8593b3-b20e-4010-bfb5-513add2780f9" />
<img width="3024" height="4032" alt="593131997-f16f22ed-d48b-4a7c-8ad4-8ddec9f013e1" src="https://github.com/user-attachments/assets/53ea5b64-3081-4373-b00a-5bfb7489379c" />
<img width="3024" height="4032" alt="593131987-819c2ea9-a394-4b10-9b12-dec3ff153cd5" src="https://github.com/user-attachments/assets/02e91d94-7e58-4954-b9cd-5852c111629d" />


**Prototyping and Evaluation**

For prototyping, I showed my friend our MVP. I intended to find out if they could name some missing features as well as
if the UI layout was optimal. The interaction unveiled that we would be better off with more components such as a leaderboard
or a currency count, because our MVP let the user bet with infinite currency. These were my main takeaways, and we ended
up implementing them in the development process later.

## Engineering

**Infrastructure**

Some of the main development tools I used this semester have been VSCode,TypeScript, React, Vite + Vitest, GitHub, and ChatGPT. 
TypeScript, React, Vite, and Vitest were honestly really helpful in the code development process, I enjoyed the suggestions and
autofills sometimes were helpful. GitHub was also very helpful, having version control was pretty irreplacable, in my opinion. There 
were so many times we had to go back and look at overwritten code or get back files that had accidentally been deleted. I think 
making these mistakes got us into the habit of frequently making commits. Some of my teammates preferred to have a GitHub extension
and found it easier to navigate. I decided to stick it out with the terminal and I honestly really enjoyed that. We got into a lot of
situations where we would have issues like accidental pushes or wrong branching and they were quite hard to navigate in the 
extension. Everyone ended up using the terminal in these situations, so it was a good skill for me to have and help with. 

As for debugging, we wound up having a lot of warnings and errors concerning imports and calls to components. ChatGPT was
very helpful in figuring out what needed to be changed, as it was usually a few lines that had to be added elsewhere.

**Self-learning**

During the tech tutorial, I was responsible for understanding and explaining JavaScript and TypeScript. This worked out well because
we chose to use typescript for all of the front-end code. In my tutorial, I explained each at a high level, compared them, and then
demonstrated how to access and set up TypeScript. This proved to be really helpful because we had to go through the process to obtain
it, and it's always nice to know the benefits of your tools.


TypeScript ended up being very useful, it would have been much more work to build our UI without it. We enjoyed how great it was with
types and catching type-related issues as we were writing our code. Some fallbacks were definitely how difficult it was to use. I was
expecting something closer to Java, so it was definitely unexpected. I hadn't used JavaScript/HTML in the past, so I think that's
why it was exceptionally hard to pick up. I watched some videos on it, but ChatGPT ended up being more helpful when it came to
explaining why we do things a certain way and what exactly each line was doing. For me personally, a major sticking point was
understanding how "div" worked and what it meant. I think it definitely took some getting used to, but looking up explanations
provided us with a lot of clarity.

**Process**

To distribute our work, my team split into two pairs for front-end and back-end development. I got to work with Sam on the front-end
while Lucas and Youssef were resposible for the back-end. Our plan was to meet for an hour on Fridays to discuss progress, but we had
an easier time working on Wednesdays so we started meeting twice a week mid semester. Not much got in the way here, everyone was
responsible and reliable, and a large portion of these meetings were productive.


This process worked out well for the most part, I think everyone stuck to their roles and there was not too much overlap, on my end
at least. We were consistent with attending meetings and any time conflicts were communicated early on through messaging. I think
what got in the way for me was the strict divide between the front and back. We would sometimes end up in situations where our merge
conflicts affected the back end, and Sam and I did not understand it well enough to work through them ourselves. Looking back, I
definitely wish we had mixed up roles for a short period earlier in the process so that we could each have some base knowledge about
the other side. That way we could have prevented the awkward time gaps that it took to fix these types of issues.

**Collaboration**

For using collaborative tools, my team and I did our best to use the chart with GitHub issues. This was a weekly activity during the
first half of the semester because the work we did was mostly remote. We would sit down on Fridays and treat it as a to-do list, 
assigning a bunch of tasks (whether that be implementation or fixing bugs). Once the second half of the semester came around, our 
work became much more collaborative. Almost all our code was written with us sitting in the same room, so we stopped using the chart
and would verbally communicate what needed to get done. I would even argue that the difficulty using the tool was caused by better 
collaboration.


For filing and resolving bugs, we also used attempted to use github issues. I opened 7 of them over the course of the semester for
things like resolving bets, creating a side view of the top bets, making the comments work, etc. For the bugs that I resolved, I
noted the issue # in my pull request. In the descriptions of the pull requests i made, I noted solving the following issues: Issue 
#43 resolved, Issue #27 complete, Issue #27 complete.

For code reviews, i often reviewed and merged my teammates' code. For some that I did not merge, I helped the person reviewing fix
merge conflicts. One notable instance of a code review was for Sam's additional betPost instance. He had tried to merge it twice.
He ended up making way too many changes to the files and deleting some of the main ones, so the conflicts were too complex to be
resolved in a web editor. The second time around, he managed to create a pull request for the same component. For some reason, it
showed no merge conflicts the second time. However, the big edits prompted me to have a conversation with him because it was the night
before the final deadline. He explained that he tried to make the code better by decluttering the betPost and splitting it into
multiple components. While I understood where he was coming from, I had concerns about things breaking the night before our deadline,
and things like testing and accessibility depended on that component. We ultimately decided as a group that it was for the best to
scrap the component to not introduce new issues, and I closed the pull request with a comment describing why I did not accept it.

**Accessible Design**

For accessibility, we added html tags for screen readers into our code as well as labels to buttons, headers, etc. Majority
of my work in design had to do with fixing UI looks after assessing the Wave score. For our MVP, we managed to get above 9,
but it came down to a 7.9 later on due to some components with gray text. I also added a bit in Registration and login that 
automatically puts the cursor in the first text box when you click on either log in or create account
