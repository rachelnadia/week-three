# install igraph; this might take a long time
# you only run this line the first time you install igraph:
install.packages('igraph')

# a lot of stuff gets downloaded and installed.

# now tell RStudio you want to use the igraph pacakge and its functions:
library('igraph')

# now let's load up the data by putting the csv files into nodes and links.
nodes <- read.csv("texasnodes.csv", header=T, as.is=T)
links <- read.csv("texaslinks.csv", header=T, as.is=T)

#examine data
head(nodes)
head(links)

nrow(nodes); length(unique(nodes$id))
# which gives the number of nodes in our data

nrow(links); nrow(unique(links[,c("source", "target")]))
# which gives the number of sources, and number of targets
# which means some people sent more than one letter, and some people received more than one letter

links <- aggregate(links[,3], links[,-3], sum)

links <- links[order(links$target, links$source),]

colnames(links)[3] <- "weight"

rownames(links) <- NULL

head(links)

# let's make a net
# notice that we are telling igraph that the network is directed, that the relationship Alice to Bob is different than Bob's to Alice (Alice is the _sender_, and Bob is the _receiver_)

# In older DH Box version of igraph in RStudio:
net <- graph.data.frame(d=links, vertices=nodes, directed=T)

# OR Newer version of igraph in desktop RStudio:
net <- graph_from_data_frame(d=links, vertices=nodes, directed=T)

# type 'net' again and run the line to see how the network is represented.
net

# let's visualizae it
plot(net, edge.arrow.size=.4,vertex.label=NA)

# two quite distinct groupings, it would appear.
---
  
  Before we jump down the rabbit hole of visualization, let's recognize right now that visualizing a network is only rarely of analytical value. The value of network analysis comes from the various questions we can now start posing of our data when it is represented as a network. In this correspondence network, who is in the centre of the web? To whom would information flow? To whom would information leak? Are there cliques or ingroups? When we identify such individuals, how does that confirm or confound our expectations of the period and place?

Many different kinds of metrics can be calculated (and the [Kateto R igraph tutorial](https://kateto.net/networks-r-igraph) will show you how) but it's always worth remembering that a metric is only meaningful for a given network when we're dealing with similar things - a network of people who write letters to one another; a network of banks that swap mortgages with one another. These are called 'one mode networks'. A network of people connected to the banks they use - a two mode network, because it connects two different kinds of things - might be useful to visualize but the metrics calculated might not be valid if the metric was designed to work on a one-mode network .

Given our correspondence network, let's imagine that 'closeness' (a measure of how central a person is) and 'betweenness' (a measure of how many different strands of the network pass through this person) are the most historically interesting. Further, we're going to try to determine if there are subgroups in our network, cliques.

```R
## the 'degree' of a node is the count of its connections. In this code chunk, we calculate degree, then make both a histogram of the counts and a plot of the network where we size the nodes proportionately to their degree. What do we learn from these two visualizations?
deg <- degree(net, mode="all")
hist(deg, breaks=1:vcount(net)-1, main="Histogram of node degree")
plot(net, vertex.size=deg*2, vertex.label = NA)
## write this info to file for safekeeping
write.csv(deg, 'degree.csv')

closepeople <- closeness(net, mode="all", weights=NA)
sort(closepeople, decreasing = T) # so that we see who is most close first
write.csv(closepeople, 'closeness.csv') # so we have it on file.

hs <- hub_score(net, weights=NA)$vector
as <- authority_score(net, weights=NA)$vector

par(mfrow=c(1,2))

# vertex.label.cex sets the size of the label; play with the sizes until you see something appealing.
plot(net, vertex.size=hs*40, vertex.label.cex =.2, edge.arrow.size=.1, main="Hubs")
plot(net, vertex.size=as*20, vertex.label = NA, edge.arrow.size=.1, main="Authorities")

cfg <- cluster_fast_greedy(as.undirected(net))

lapply(cfg, function(x) write.table( data.frame(x), 'cfg.csv'  , append= T, sep=',' ))

plot(cfg, net, vertex.size = 1, vertex.label.cex =.2, edge.arrow.size=.1, main="Communities")

l1 <- layout_with_fr(net)

plot(cfg, net, layout=l1, vertex.size = 1, vertex.label.cex =.2, edge.arrow.size=.1, main="Communities")