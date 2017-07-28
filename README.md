## Ant Colony Optimization
### with node.js

ACO is one of the most konwn Optimization and TSP Algorithm.

So in my masters degree i had to implement it for Sorting my multi-dimentional Graphs.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```javascript
var _ = require('underscore');
module.exports = function(dists, dimensions) {

    var __ = {
        numAnts: 20,
        maxTime: 100,
        numCities: dists.length,
        alpha: 1, // influence of pheromone on direction
        beta: 2,
        rho: 0.1, // pheromone decrease factor
        Q: 2.0,
        maxDistance: _.chain(dists).map(function(dist) {
            return _.max(dist);
        }).max().value()
    };

    var chaine = '';
    return {
        ACO: function() {

            try {

                chaine += "Number cities in problem = " + __.numCities;

                chaine += "\nNumber ants = " + __.numAnts;
                chaine += "\nMaximum time = " + __.maxTime;

                chaine += "\nAlpha (pheromone influence) = " + __.alpha;
                chaine += "\nBeta (local node influence) = " + __.beta;
                chaine += "\nRho (pheromone evaporation coefficient) = " + __.rho;
                chaine += "\n__.Q (pheromone deposit factor) = " + __.Q;

                chaine += "\nInitialing dummy graph distances";


                chaine += "\nInitialing ants to random trails\n";
                var ants = InitAnts(__.numAnts, __.numCities); // initialize ants to random trails
                ShowAnts(ants, dists);
                var bestTrail = BestTrail(ants, dists); // determine the best initial trail
                var bestLength = Length(bestTrail, dists); // the length of the best trail
                var pheromones = InitPheromones(__.numCities);

                var time = 0;

                while (time < __.maxTime) {
                    UpdateAnts(ants, pheromones, dists);
                    UpdatePheromones(pheromones, ants, dists);

                    var currBestTrail = BestTrail(ants, dists);
                    var currBestLength = Length(currBestTrail, dists);

                    if (currBestLength < bestLength && CheckTrail(currBestTrail) == true) {
                        bestLength = currBestLength;
                        bestTrail = currBestTrail;
                    }
                    ++time;
                }

                //console.log("BestTrail"+bestTrail+" >> bestLength"+bestLength);
                console.log("\nBestTrail " + bestTrail + " >> bestLength " + bestLength);
                // console.log("result :\n"+chaine);
                // console.log(chaine);
                return bestTrail;
            } catch (err) {
                console.log(err.message);
            }
        },
        glouton: function() {
            try {
                chaine += "Number cities in problem = " + __.numCities;
                var chemain = new Array();
                chemain[__.numCities + 1] = __.maxDistance * __.numCities * 10;
                var temporaire = new Array();

                for (var s = 0; s < __.numCities; s++) {
                    temporaire = [];
                    temporaire[0] = s;
                    temporaire[__.numCities + 1] = 0;
                    for (var i = 0; i < __.numCities - 1; i++) {
                        temporaire[i + 1] = NextEdge(dists[temporaire[i]], temporaire);
                        temporaire[__.numCities + 1] += dists[temporaire[i]][temporaire[i + 1]];
                    }

                    if (temporaire[__.numCities + 1] < chemain[__.numCities + 1]) {
                        for (var t = 0; t <= __.numCities + 1; t++) {
                            chemain[t] = temporaire[t];
                        }
                    }
                }
                chaine += "\nBestTrail " + chemain;
                console.log(chaine);
                return chemain.slice(0, __.numCities);
            } catch (err) {
                console.log(err.message);
            }
        },
        branchandbound: function() {
            var TSP_BnB = require('./Branch&Bound');
            var result = new TSP_BnB(dimensions, dists, __.maxDistance * __.numCities * 10);
            return result;
        }

    };

    function NextEdge(trail, chemain) {
        var next = __.maxDistance * __.numCities * 10;
        var index = 0;
        for (var i = 0; i < trail.length; ++i) {
            if (trail[i] < next && chemain.indexOf(i) == -1) {
                next = trail[i];
                index = i;
            }
        }

        return index;
    }
    //------------------------------------------------------------------------
    function InitAnts(numAnts, numCities) {
        var ants = new Array(numAnts);
        for (var k = 0; k < numAnts; ++k) {

            ants[k] = new Array(numCities);
            var start = Math.floor((Math.random() * numCities) + 0);
            ants[k] = RandomTrail(start, numCities);
        }

        return ants;
    }
    //-------------------------------------------------------------------------
    function RandomTrail(start, numCities) // helper for InitAnts
    {
        var trail = new Array(numCities);
        var transit = false;

        for (var i = 0; i < numCities; ++i) {
            trail[i] = i;
        } // sequential

        for (var i = 0; i < numCities; ++i) // Fisher-Yates shuffle
        {
            var r = Math.floor(Math.random() * (numCities - i) + i);

            var tmp = trail[r];
            trail[r] = trail[i];
            trail[i] = tmp;

        }

        var idx = IndexOfTarget(trail, start); // put start at [0]
        var temp = trail[0];
        trail[0] = trail[idx];
        trail[idx] = temp;

        return trail;
    }
    //-------------------------------------------------------------------------
    function CheckTrail(trail) {
        var i = 0;
        var transit = true;
        /* while ( i < trail.length-1 && transit == true)
     {  

       if( dists[trail[i]][trail[i+1]] < 0 )
           transit = false;

        i++;
     } */
        return transit;
    }
    //-----------------------------------------------------------------------
    function IndexOfTarget(trail, target) // helper for RandomTrail
    {
        for (var i = 0; i < trail.length; ++i) {
            if (trail[i] == target) {
                return i;
            }
        }
        console.log("Target not found in IndexOfTarget");
    }
    //----------------------------------------------------------------------
    function Length(trail, dists) // total length of a trail
    {
        var result = 0.0;
        for (var i = 0; i < trail.length - 1; ++i)
            result += Distance(trail[i], trail[i + 1], dists);
        return result;
    }
    //---------------------------------------------------------------------
    function BestTrail(ants, dists) // best trail has shortest total length
    {
        var bestLength = Length(ants[0], dists);
        var idxBestLength = 0;
        var k;
        for (k = 1; k < ants.length; ++k) {
            var len = Length(ants[k], dists);
            if (len < bestLength) {
                bestLength = len;
                idxBestLength = k;
            }
        }
        var numCities = ants[0].length;

        return ants[k - 1];
    }
    //-------------------------------------------------------------------
    function InitPheromones(numCities) {
        var pheromones = new Array(numCities);
        for (var i = 0; i < numCities; ++i)
            pheromones[i] = new Array(numCities);
        for (var i = 0; i < pheromones.length; ++i)
            for (var j = 0; j < pheromones[i].length; ++j)
                pheromones[i][j] = 0.01; // otherwise first call to UpdateAnts -> BuiuldTrail -> NextNode -> MoveProbs => all 0.0 => throws
        return pheromones;
    }
    //------------------------------------------------------------------
    function UpdateAnts(ants, pheromones, dists) {
        var numCities = pheromones.length;
        for (var k = 0; k < ants.length; ++k) {
            var start = Math.floor((Math.random() * numCities) + 0);
            var newTrail = BuildTrail(k, start, pheromones, dists);
            ants[k] = newTrail;
        }
    }
    //------------------------------------------------------------------
    function BuildTrail(k, start, pheromones, dists) {
        var numCities = pheromones.length;
        var trail = new Array(numCities);
        var visited = new Array(numCities);
        trail[0] = start;
        visited[start] = true;
        for (var i = 0; i < numCities - 1; ++i) {
            var cityX = trail[i];
            var next = NextCity(k, cityX, visited, pheromones, dists);
            trail[i + 1] = next;
            visited[next] = true;
        }
        return trail;
    }
    //--------------------------------------------------------------------
    function NextCity(k, cityX, visited, pheromones, dists) {
        // for ant k (with visited[]), at nodeX, what is next node in trail?
        var probs = MoveProbs(k, cityX, visited, pheromones, dists);
        var cumul = new Array(probs.length + 1);

        for (var i = 0; i < probs.length + 1; ++i)
            cumul[i] = 0;

        for (var i = 0; i < probs.length; ++i)
            cumul[i + 1] = cumul[i] + probs[i]; // consider setting cumul[cuml.Length-1] to 1.00  

        var p = Math.random();

        for (var i = 0; i < cumul.length - 1; ++i)
            if (p >= cumul[i] && p < cumul[i + 1])
                return i;


        throw new Exception("Failure to return valid city in NextCity");
    }
    //--------------------------------------------------------------------
    function MoveProbs(k, cityX, visited, pheromones, dists) {
        // for ant k, located at nodeX, with visited[], return the prob of moving to each city
        var numCities = pheromones.length;
        var taueta = new Array(numCities); // inclues cityX and visited cities
        var sum = 0.0; // sum of all tauetas
        for (var i = 0; i < taueta.length; ++i) // i is the adjacent city
        {
            if (i == cityX)
                taueta[i] = 0.0; // prob of moving to self is 0
            else if (visited[i] == true)
                taueta[i] = 0.0; // prob of moving to a visited city is 0
            else {
                taueta[i] = Math.pow(pheromones[cityX][i], __.alpha) * Math.pow((1.0 / Distance(cityX, i, dists)), __.beta); // could be huge when pheromone[][] is big
                if (taueta[i] < 0.0001)
                    taueta[i] = 0.0001;
                else if (taueta[i] > (Number.MAX_VALUE / (numCities * 100)))
                    taueta[i] = Number.MAX_VALUE / (numCities * 100);

            }
            sum += taueta[i];
        }

        var probs = new Array(__.numCities);
        for (var i = 0; i < probs.length; ++i) {
            if (sum == 0)
                probs[i] = 0;
            else
                probs[i] = taueta[i] / sum; // big trouble if sum = 0.0
        }
        return probs;
    }
    //------------------------------------------------------------------
    function UpdatePheromones(pheromones, ants, dists) {
        for (var i = 0; i < pheromones.length; ++i) {
            for (var j = 0; j < pheromones[i].length; ++j) {
                for (var k = 0; k < ants.length; ++k) {
                    var length = Length(ants[k], dists); // length of ant k trail
                    var decrease = (1.0 - __.rho) * pheromones[i][j];
                    var increase = 0.0;
                    if (EdgeInTrail(i, j, ants[k]) == true) increase = (__.Q / length);

                    pheromones[i][j] = decrease + increase;

                    if (pheromones[i][j] < 0.0001)
                        pheromones[i][j] = 0.0001;
                    else if (pheromones[i][j] > 100000.0)
                        pheromones[i][j] = 100000.0;
                }
            }
        }
    }
    //----------------------------------------------------------------------
    function EdgeInTrail(cityX, cityY, trail) {
        // are cityX and cityY adjacent to each other in trail[]?
        var lastIndex = trail.length - 1;
        var idx = IndexOfTarget(trail, cityX);

        if (idx == 0 && trail[1] == cityY) return true;
        else if (idx == 0 && trail[lastIndex] == cityY) return true;
        else if (idx == 0) return false;
        else if (idx == lastIndex && trail[lastIndex - 1] == cityY) return true;
        else if (idx == lastIndex && trail[0] == cityY) return true;
        else if (idx == lastIndex) return false;
        else if (trail[idx - 1] == cityY) return true;
        else if (trail[idx + 1] == cityY) return true;
        else return false;
    }
    //-----------------------------------------------------------------------
    function Display(trail) {
        for (var i = 0; i < trail.Length; ++i) {
            chaine += "\n" + trail[i] + " ";
            if (i > 0 && i % 20 == 0) chaine += " ";
        }
        chaine += "\n";
    }
    //-----------------------------------------------------------------------
    function Distance(cityX, cityY, dists) {
        return dists[cityX][cityY];
    }
    //-----------------------------------------------------------------------
    function ShowAnts(ants, dists) {
        for (var i = 0; i < ants.length; ++i) {
            chaine += "\n" + i + ": [ ";

            for (var j = 0; j < __.numCities; ++j)
                chaine += ants[i][j] + "  ";

            chaine += " ] len = ";
            var len = Length(ants[i], dists);
            chaine += len;
            chaine += "\n";
        }
    }

    //-----------------------------------------------------------------------
    function Display(pheromones) {
        for (var i = 0; i < pheromones.length; ++i) {
            chaine += i + ": ";
            for (var j = 0; j < pheromones[i].length; ++j) {
                chaine += pheromones[i][j] + " ";
            }
            chaine += '\n';
        }

    }
};

```

