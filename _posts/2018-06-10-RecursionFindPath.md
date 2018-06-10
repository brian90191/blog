---
title: 使用JS及遞迴尋找兩點的所有路徑
tags: Recursion
categories: Javascript
---

* TOC
{:toc}

最近實做了一個功能是有關網路圖的呈現，DB使用了NEO4J然後最終資料會於前端呈現，前端則使用了強大的Vis JS來做，有個功能則是希望可以找到某兩點的所有路徑，原本是打算直接使用NEO4J的路徑查詢語法來實做，但發現每次都跑到後端處理再丟回前段實在太沒有效率，所以就寫了遞回來做路徑查詢。

線上展示: http://jsfiddle.net/dbsgy9o2/

範例的樹狀圖會長這樣

![sample-relation][sample-relation]

```Javascript
//sample data
var connections = [
    {
        from: "Node_1",
        to: "Node_2"
    },
    {
        from: "Node_1",
        to: "Node_3"
    },
    {
        from: "Node_2",
        to: "Node_4"
    },
    {
        from: "Node_3",
        to: "Node_5"
    },
    {
        from: "Node_3",
        to: "Node_7"
    },
    {
        from: "Node_3",
        to: "Node_11"
    },
    {
        from: "Node_4",
        to: "Node_6"
    },
    {
        from: "Node_6",
        to: "Node_8"
    },
    {
        from: "Node_8",
        to: "Node_9"
    },
     {
        from: "Node_5",
        to: "Node_9"
    },
    {
        from: "Node_5",
        to: "Node_10"
    },
    {
        from: "Node_5",
        to: "Node_11"
    },
    {
        from: "Node_10",
        to: "Node_11"
    },
];

//send request too get result
var paths_result = findPaths(connections, "Node_1", "Node_11");
console.log(paths_result);

//find paths for connections data
function findPaths(connections, start, end) {
  //first, build a tree, for loads of connections
  var tree = buildTree(
    connections,
    findConnection(connections, start, 0),
    null
  );

  //make the tree into all the different paths
  var allPaths = buildPaths(tree, []);

  //pare down the paths to ones that fit the start and end nodes
  var goodPaths = verifyPaths(allPaths, start, end);

  //reformat
  return paths_format(goodPaths);
}

//runs through the connections array and returns an array of elements
function findConnection(connections, source, prev_source) {
  return connections.filter(
    function(conn){
        return conn.from === source && conn.to !== prev_source;
    }
  );
}

//returns an array that contains a tree
function buildTree(connections, nodes, parent) {
  return nodes.map(function(node){
    //add the node's paren to the node
    node.parent = parent;

    //find any other connections for current target node
    node.path = findConnection(connections, node.to, node.from);

    //if there is child nodes found, then build sub-tree
    if (node.path && node.path.length > 0 ) {
      buildTree(connections, node.path, {
        from: node.from,
        to: node.to,
        parent: node.parent
      });
    }

    return node;
  });
}

//turns the "tree" into an array of all of the paths from beginning to end
function buildPaths(tree, prev) {
  tree.forEach(function(step){
    if (step.path.length > 0) {
      prev.push(step);
      buildPaths(step.path, prev);
    } else {
      prev.push(step);
    }
  });
  return prev;
}

// filter out the paths that don't match the start and end
function verifyPaths(paths, start, end) {
  return paths.filter(function(path){
      return path.to === end
  }).filter(function(path){
    while (path.parent) {
      path = path.parent;
    }
    return path.from === start;
  });
}

//format paths array
function paths_format(arr) {
  return arr.map(function(el){
    var paths_temp = [];
    paths_temp.unshift({
      from: el.from,
      to: el.to
    });
    while (el.parent) {
      el = el.parent;
      paths_temp.unshift({
        from: el.from,
        to: el.to
      });
    }
    return paths_temp;
  });
}
```

Try it!

[sample-relations]: {{"/2018-06-10-RecursionFindPath/sample-relations.png" | prepend: site.imgrepo }}
