# league-stat-analysis
# league-stat-analysis
<section id="introduction">
 <h2>Introduction</h2>


 <p>
   Professional League of Legends matches move very quickly. A single game can be decided by a
   handful of key moments in the first 15 minutes way long before most fans can tell who's
   actually going to win. Commentators are constantly reading early gold leads, kill counts,
   and objective control to guess at a team's odds, but how much do those early signals
   <em>really</em> tell us?
 </p>


 <p>
   This project uses professional match data from
   <a href="https://oracleselixir.com/" target="_blank" rel="noopener noreferrer">Oracle's Elixir</a>,
   one of the most widely used sources of League of Legends esports statistics, covering the
   2026 competitive season. Each row represents either a players performance or overall team's performance in a single game.
   This means that for every game there are 12 rows (5 players on each team and 2 overall team rows).
 </p>


 <p>
   <strong>The question at the center of this project: Can we predict whether a team will win
   or lose a match using only the information available at the 15-minute mark?</strong>
 </p>


 <p>
   This is important because if early-game stats can reliably predict the outcome of a game as a whole,
   it means the first half of the game effectively decides the match.
   If they can't, it means League of Legends games are more volatile
   than they appear.
 </p>


 <p>
   <strong>Dataset size:</strong> After cleaning, the dataset contains <strong>15888 rows</strong>,
   each representing one team's performance in one professional match.
 </p>


 <h3>Columns used to answer this question</h3>


 <table>
   <thead>
     <tr>
       <th>Column</th>
       <th>What it represents</th>
     </tr>
   </thead>
   <tbody>
     <tr>
       <td><code>result</code> / <code>result_label</code></td>
       <td>Whether the team won or lost the game (what we're trying to predict)</td>
     </tr>
     <tr>
       <td><code>firstPick</code></td>
       <td>Whether the team had first pick during the champion draft</td>
     </tr>
     <tr>
       <td><code>firstblood</code></td>
       <td>Whether the team secured the game's first kill</td>
     </tr>
     <tr>
       <td><code>firstdragon</code></td>
       <td>Whether the team secured the game's first dragon</td>
     </tr>
     <tr>
       <td><code>firstherald</code></td>
       <td>Whether the team secured the game's first Rift Herald</td>
     </tr>
     <tr>
       <td><code>goldat10</code> / <code>goldat15</code></td>
       <td>Team's total gold at 10 and 15 minutes</td>
     </tr>
     <tr>
       <td><code>xpat10</code> / <code>xpat15</code></td>
       <td>Team's total experience at 10 and 15 minutes</td>
     </tr>
     <tr>
       <td><code>csat10</code> / <code>csat15</code></td>
       <td>Team's total creep score (minions + jungle monsters) at 10 and 15 minutes</td>
     </tr>
     <tr>
       <td><code>killsat10</code> / <code>killsat15</code></td>
       <td>Team kills by 10 and 15 minutes</td>
     </tr>
     <tr>
       <td><code>assistsat10</code> / <code>assistsat15</code></td>
       <td>Team assists by 10 and 15 minutes</td>
     </tr>
     <tr>
       <td><code>deathsat10</code> / <code>deathsat15</code></td>
       <td>Team deaths by 10 and 15 minutes</td>
     </tr>
     <tr>
       <td><code>gamelength</code></td>
       <td>Total game length in seconds (used later in the fairness analysis)</td>
     </tr>
   </tbody>
 </table>
</section>
