# league-stat-analysis
<section id="introduction">
 <h2>Introduction</h2>


 <p>
   Professional League of Legends matches move very quickly. A single game can be decided by a
   handful of key moments in the first 15 minutes way before most fans can tell who's
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
<section id="data-cleaning-and-eda">
  <h2>Data Cleaning and Exploratory Data Analysis</h2>

  <h3>Data Cleaning</h3>

  <p>
    <strong>1. Filtering to team-level rows.</strong><br>
    Oracle's Elixir records one row per <em>player</em> per game, plus a separate aggregated
    row per <em>team</em> per game (identified by <code>position == 'team'</code>). Since this
    project is about predicting team outcomes rather than individual performance, I filtered
    down to only the team-level rows:
  </p>
  <pre><code>teams = lol[lol['position'] == 'team'].drop('position', axis=1)</code></pre>
  <p>
    This immediately removes about 5/6 of the raw rows (5 players + 1 team row per team per
    game).
  </p>

  <p>
    <strong>2. Filtering to complete data.</strong><br>
    Oracle's Elixir flags each game with a <code>datacompleteness</code> field, since some
    matches are scraped from incomplete broadcast/replay data (missing timestamps, missing
    draft info, etc.). I kept only rows marked <code>'complete'</code>:
  </p>
  <pre><code>teams = teams[teams['datacompleteness'] == 'complete'].drop('datacompleteness', axis=1)</code></pre>
  <p>
    <strong>3. Selecting relevant columns.</strong><br>
    The raw dataset has 70+ columns covering everything from bans/picks to full-game damage
    and vision stats. I narrowed this down to the columns relevant to my prediction question.
  </p>

  <p>
    <strong>4. Converting boolean-like columns.</strong><br>
    Columns like <code>firstPick</code>, <code>firstblood</code>, <code>firstdragon</code>,
    <code>firstherald</code>, <code>firsttower</code>, <code>firstmidtower</code>, and
    <code>firsttothreetowers</code> are stored as 0/1 floats in the raw CSV but are
    conceptually true/false flags, so I cast them to booleans:
  </p>
  <pre><code>cleaned[[...]] = cleaned[[...]].astype(bool)</code></pre>
  <h3>Cleaned DataFrame</h3>

  <p>Below is the first few rows of the cleaned dataset:</p>

  <table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>result</th>
      <th>gameid</th>
      <th>teamname</th>
      <th>firstPick</th>
      <th>gamelength</th>
      <th>kills</th>
      <th>deaths</th>
      <th>assists</th>
      <th>doublekills</th>
      <th>triplekills</th>
      <th>quadrakills</th>
      <th>pentakills</th>
      <th>firstblood</th>
      <th>team kpm</th>
      <th>ckpm</th>
      <th>firstdragon</th>
      <th>dragons</th>
      <th>elders</th>
      <th>firstherald</th>
      <th>heralds</th>
      <th>void_grubs</th>
      <th>barons</th>
      <th>firsttower</th>
      <th>towers</th>
      <th>firstmidtower</th>
      <th>firsttothreetowers</th>
      <th>turretplates</th>
      <th>inhibitors</th>
      <th>damagetochampions</th>
      <th>dpm</th>
      <th>damagetakenperminute</th>
      <th>damagemitigatedperminute</th>
      <th>damagetotowers</th>
      <th>wardsplaced</th>
      <th>wpm</th>
      <th>wardskilled</th>
      <th>wcpm</th>
      <th>controlwardsbought</th>
      <th>visionscore</th>
      <th>vspm</th>
      <th>earned gpm</th>
      <th>goldspent</th>
      <th>gspd</th>
      <th>gpr</th>
      <th>minionkills</th>
      <th>monsterkills</th>
      <th>cspm</th>
      <th>goldat10</th>
      <th>xpat10</th>
      <th>csat10</th>
      <th>killsat10</th>
      <th>assistsat10</th>
      <th>deathsat10</th>
      <th>goldat15</th>
      <th>xpat15</th>
      <th>csat15</th>
      <th>killsat15</th>
      <th>assistsat15</th>
      <th>deathsat15</th>
      <th>goldat20</th>
      <th>xpat20</th>
      <th>csat20</th>
      <th>killsat20</th>
      <th>assistsat20</th>
      <th>deathsat20</th>
      <th>goldat25</th>
      <th>xpat25</th>
      <th>csat25</th>
      <th>killsat25</th>
      <th>assistsat25</th>
      <th>deathsat25</th>
      <th>result_label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10</th>
      <td>0</td>
      <td>LOLTMNT05_171038</td>
      <td>GMBLERS Esports</td>
      <td>True</td>
      <td>1756</td>
      <td>22</td>
      <td>29</td>
      <td>28</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>0.75</td>
      <td>1.74</td>
      <td>True</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>3.0</td>
      <td>True</td>
      <td>True</td>
      <td>22.0</td>
      <td>0.0</td>
      <td>93473</td>
      <td>3193.84</td>
      <td>4230.75</td>
      <td>2944.13</td>
      <td>18382.0</td>
      <td>81</td>
      <td>2.77</td>
      <td>38</td>
      <td>1.30</td>
      <td>12</td>
      <td>241</td>
      <td>8.23</td>
      <td>1270.46</td>
      <td>53361</td>
      <td>-0.02</td>
      <td>-0.18</td>
      <td>707.0</td>
      <td>180</td>
      <td>30.31</td>
      <td>17079.0</td>
      <td>19437.0</td>
      <td>309.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>29607.0</td>
      <td>32613.0</td>
      <td>482.0</td>
      <td>12.0</td>
      <td>11.0</td>
      <td>13.0</td>
      <td>40952.0</td>
      <td>45397.0</td>
      <td>642.0</td>
      <td>18.0</td>
      <td>20.0</td>
      <td>19.0</td>
      <td>49357.0</td>
      <td>55550.0</td>
      <td>795.0</td>
      <td>20.0</td>
      <td>22.0</td>
      <td>20.0</td>
      <td>Loss</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1</td>
      <td>LOLTMNT05_171038</td>
      <td>EKO Esports</td>
      <td>False</td>
      <td>1756</td>
      <td>29</td>
      <td>22</td>
      <td>51</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>0.99</td>
      <td>1.74</td>
      <td>False</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>False</td>
      <td>7.0</td>
      <td>False</td>
      <td>False</td>
      <td>26.0</td>
      <td>1.0</td>
      <td>98600</td>
      <td>3369.02</td>
      <td>4119.19</td>
      <td>3755.43</td>
      <td>28875.0</td>
      <td>86</td>
      <td>2.94</td>
      <td>28</td>
      <td>0.96</td>
      <td>17</td>
      <td>220</td>
      <td>7.52</td>
      <td>1464.46</td>
      <td>54200</td>
      <td>0.02</td>
      <td>0.18</td>
      <td>718.0</td>
      <td>203</td>
      <td>31.47</td>
      <td>17617.0</td>
      <td>20161.0</td>
      <td>346.0</td>
      <td>6.0</td>
      <td>2.0</td>
      <td>5.0</td>
      <td>28500.0</td>
      <td>31959.0</td>
      <td>496.0</td>
      <td>13.0</td>
      <td>11.0</td>
      <td>12.0</td>
      <td>38716.0</td>
      <td>45019.0</td>
      <td>648.0</td>
      <td>19.0</td>
      <td>24.0</td>
      <td>18.0</td>
      <td>47602.0</td>
      <td>56563.0</td>
      <td>824.0</td>
      <td>20.0</td>
      <td>26.0</td>
      <td>20.0</td>
      <td>Win</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1</td>
      <td>LOLTMNT05_172024</td>
      <td>Deacoy</td>
      <td>True</td>
      <td>2243</td>
      <td>14</td>
      <td>15</td>
      <td>34</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>0.37</td>
      <td>0.78</td>
      <td>False</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>True</td>
      <td>10.0</td>
      <td>True</td>
      <td>True</td>
      <td>40.0</td>
      <td>1.0</td>
      <td>134685</td>
      <td>3602.81</td>
      <td>4632.57</td>
      <td>4715.31</td>
      <td>34940.0</td>
      <td>94</td>
      <td>2.51</td>
      <td>50</td>
      <td>1.34</td>
      <td>19</td>
      <td>326</td>
      <td>8.72</td>
      <td>1240.04</td>
      <td>63138</td>
      <td>0.04</td>
      <td>0.67</td>
      <td>1020.0</td>
      <td>259</td>
      <td>34.21</td>
      <td>16659.0</td>
      <td>20787.0</td>
      <td>365.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>24401.0</td>
      <td>32478.0</td>
      <td>555.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>33495.0</td>
      <td>45303.0</td>
      <td>773.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>7.0</td>
      <td>43609.0</td>
      <td>58120.0</td>
      <td>986.0</td>
      <td>3.0</td>
      <td>4.0</td>
      <td>9.0</td>
      <td>Win</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0</td>
      <td>LOLTMNT05_172024</td>
      <td>Zena Esports</td>
      <td>False</td>
      <td>2243</td>
      <td>15</td>
      <td>14</td>
      <td>32</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>0.40</td>
      <td>0.78</td>
      <td>True</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>2.0</td>
      <td>False</td>
      <td>False</td>
      <td>15.0</td>
      <td>0.0</td>
      <td>140533</td>
      <td>3759.24</td>
      <td>4234.99</td>
      <td>5020.63</td>
      <td>14396.0</td>
      <td>111</td>
      <td>2.97</td>
      <td>35</td>
      <td>0.94</td>
      <td>14</td>
      <td>303</td>
      <td>8.11</td>
      <td>1052.42</td>
      <td>60818</td>
      <td>-0.04</td>
      <td>-0.67</td>
      <td>920.0</td>
      <td>267</td>
      <td>31.75</td>
      <td>15957.0</td>
      <td>20168.0</td>
      <td>327.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>25661.0</td>
      <td>34475.0</td>
      <td>522.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>2.0</td>
      <td>35112.0</td>
      <td>47753.0</td>
      <td>739.0</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>2.0</td>
      <td>43510.0</td>
      <td>58917.0</td>
      <td>906.0</td>
      <td>9.0</td>
      <td>15.0</td>
      <td>3.0</td>
      <td>Loss</td>
    </tr>
    <tr>
      <th>34</th>
      <td>0</td>
      <td>LOLTMNT05_171043</td>
      <td>Axolotl</td>
      <td>True</td>
      <td>1829</td>
      <td>8</td>
      <td>23</td>
      <td>12</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>False</td>
      <td>0.26</td>
      <td>1.02</td>
      <td>False</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>True</td>
      <td>5.0</td>
      <td>True</td>
      <td>True</td>
      <td>32.0</td>
      <td>0.0</td>
      <td>78802</td>
      <td>2585.08</td>
      <td>3275.59</td>
      <td>3745.39</td>
      <td>30180.0</td>
      <td>98</td>
      <td>3.21</td>
      <td>36</td>
      <td>1.18</td>
      <td>20</td>
      <td>254</td>
      <td>8.33</td>
      <td>1112.90</td>
      <td>49165</td>
      <td>-0.13</td>
      <td>-0.52</td>
      <td>820.0</td>
      <td>182</td>
      <td>32.87</td>
      <td>16097.0</td>
      <td>19344.0</td>
      <td>327.0</td>
      <td>2.0</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>26963.0</td>
      <td>33030.0</td>
      <td>518.0</td>
      <td>6.0</td>
      <td>8.0</td>
      <td>9.0</td>
      <td>35962.0</td>
      <td>43496.0</td>
      <td>688.0</td>
      <td>7.0</td>
      <td>11.0</td>
      <td>9.0</td>
      <td>44694.0</td>
      <td>53434.0</td>
      <td>866.0</td>
      <td>8.0</td>
      <td>12.0</td>
      <td>14.0</td>
      <td>Loss</td>
    </tr>
  </tbody>
</table>
</section>
