---
---
<style>

.video circle {
/*   fill: steelblue; */
  fill: black;
  stroke: black;
  stroke-width: .1px;
}
.selected circle {
  fill: red;
}
.aura circle {
  fill: lightsteelblue;
/*   stroke: black; */
/*   stroke-width: .1px; */
}

.video text {
  fill: black;
  opacity:.70
}

.axis line {
  fill: none;
/*   stroke: #ddd; */
  stroke: #fef;
  shape-rendering: crispEdges;
  vector-effect: non-scaling-stroke;
}

.line {
  fill: none;
  stroke: brown;
  stroke-width: 0.9px;
  opacity: 0.15;
}
.noline {
  fill: none;
  stroke: none;
}
#controlpanel {
  position:absolute;
  float:right;
  width:180px; height:150px;
  right:1%;top:1%;
  border-style:solid;border-width:thin;
  opacity:0.5;
  background-color:lightsteelblue;
  box-shadow:-6px 6px 3px #888888;
}
#clusterpanel {
  position:absolute;
  float:left;
  width:34%; height:40%;
  left:1%;bottom:3%;
  border-style:solid;border-width:thin;
  opacity:0.5;
  background-color:lightsteelblue;
  box-shadow:6px 6px 3px #888888;
/*   display:none; */
}

</style>
<body>

<!-- <div id="video_player" style="width: 300px; height:500px;"> -->
<div id="container" style="
    width: 95%;
    height: 90%;
    margin-left: 2.5%;
    margin-right: auto;
    margin-top: 2%;
    border-width: 1px;
    border-style: solid;
    border-radius: 10px;
    border-color: #008040;
    overflow: hidden;
    position:relative;
    ">
    <div id="player" style="width: 34%; height:100%;margin-top:2%;margin-left:2%;margin-right:2%;float:left"></div>
    <div id="navigator" style="width: 62%; height:800px;float:left;position:relative;">
    </div>
    <div id="controlpanel">
        <div id="form" style="margin:15px;">
            <label for="show_clusters"> <input id="show_clusters" type="checkbox">show clusters</label><br>
            <label for="show_tag_clouds"> <input id="show_tag_clouds" type="checkbox">show tag clouds</label><br>
            <label for="show_videos"> <input id="show_videos" type="checkbox">show videos</label>
            <!--
            show player, autoplay
            show tags
            load different data files

            filter/search interface
            auras
            -->
        </div>
    </div>
    <div id="clusterpanel">

    </div>
</div>

<script src="http://d3js.org/d3.v3.min.js"></script>
<script src="http://d3js.org/colorbrewer.v1.min.js"></script>
<script src="/lib/underscore-min.js"></script>
<script src="/lib/d3.layout.cloud.js"></script>
<script src="/lib/jquery-2.1.1.min.js"></script>
<script>
function set_checked(selector, state) {$(selector).prop("checked", state)}
function set_checked_changed_functions(selector, checked_fn, unchecked_fn) {
    $(selector).change(function() {
        if($(this).is(':checked')) checked_fn()
        else unchecked_fn()});}

function set_show_clusters_functions(checked_fn, unchecked_fn) {
    set_checked_changed_functions('input#show_clusters', checked_fn, unchecked_fn)}
function set_show_clusters_checked(state) {
    set_checked('input#show_clusters', state)}

function set_show_tag_clouds_functions(checked_fn, unchecked_fn) {
    set_checked_changed_functions('input#show_tag_clouds', checked_fn, unchecked_fn)}
function set_show_tag_clouds_checked(state) {
    set_checked('input#show_tag_clouds', state)}

function set_show_videos_functions(checked_fn, unchecked_fn) {
    set_checked_changed_functions('input#show_videos', checked_fn, unchecked_fn)}
function set_show_videos_checked(state) {
    set_checked('input#show_videos', state)}

// GET parameter retrieval
function getQueryVariable(variable) {
       var query = window.location.search.substring(1);
       var vars = query.split("&");
       for (var i=0;i<vars.length;i++) {
               var pair = vars[i].split("=");
               if(pair[0] == variable){return pair[1];}
       }
       return(false);
}


function draw_grid_lines(container, width, height, scaling_factor) {
    container.append("g")
        .attr("class", "x axis")
      .selectAll("line")
        .data(d3.range(0, width, scaling_factor))
      .enter().append("line")
        .attr("x1", function(d) { return d; })
        .attr("y1", 0)
        .attr("x2", function(d) { return d; })
        .attr("y2", height);

    container.append("g")
        .attr("class", "y axis")
      .selectAll("line")
        .data(d3.range(0, height, scaling_factor))
      .enter().append("line")
        .attr("x1", 0)
        .attr("y1", function(d) { return d; })
        .attr("x2", width)
        .attr("y2", function(d) { return d; });
}

var navi = d3.select("#navigator")
navi_height = (navi.property('clientHeight'))
navi_width = (navi.property('clientWidth'))

function constrain_to_aspect_ratio(width, height) {
  target_ratio = 0.5625
  actual_ratio = width / height
  if (actual_ratio > target_ratio) {
    return [width, width * target_ratio]
  } else {
    return [height / target_ratio, height]
  }
}
var player = d3.select("#player")
size = constrain_to_aspect_ratio(player.property('clientWidth'), player.property('clientHeight'))
player_width = size[0]
player_height = size[1]

var video_player =
    player.append("div")
        .attr("width", player_width)
        .attr("height", player_height)
        .style('display', 'block')
var video_title =
    player.append("span")
          .style('font-size', '22px')
          .style('font-family', 'arial, sans-serif')
          .style('padding-bottom', '9px')
          .style('padding-left', '20px')
          .style('padding-right', '20px')
          .style('padding-top', '15px')
          .style('display', 'block')
var video_description = player.append("span")
// var video_cluster = player.append("span")
function show_video(video) {
  var autoplay = 0
  video_url = '//www.youtube.com/embed/' + video['video.id']
  video_player.html('<iframe width="' + player_width + '" height="' + player_height + '" src="' + video_url + '?rel=0&autoplay=' + autoplay + '" frameborder="0" allowfullscreen></iframe>')
  video_title.text(video['video.title'])
//   video_cluster.text('cluster id:' + video['cluster'])
}
var cluster_panel = $("#clusterpanel")
cluster_panel.append("<span style='font-weight:bold;padding:1%;display:block' class='cluster_panel_title'></span>")
cluster_panel.append("<div class='tag_cloud' id='tag_cloud_1' style='padding:2%;float:left;width:44%'></div>")
cluster_panel.append("<div class='tag_cloud' id='tag_cloud_2' style='padding:2%;float:right;width:44%'></div>")
var cluster_title = $('.cluster_panel_title')
var tags1 = $('#tag_cloud_1')
var tags2 = $('#tag_cloud_2')
// could use a tabbed view here to show lots of detail

// var cluster_tags1 = d3.select(cluster_panel).append("span")
// var cluster_tags2 = d3.select(cluster_panel).append("span")
// var cluster_tags1 = $(cluster_panel).append("span").text($("hey!"))
function format_tags(tags) {
  result = ""
  for (i=0; i!=Math.min(7, tags.length); i++) {
    text = tags[i][0].join(" ")
    rating = tags[i][1]
    result += "<span>" + text + "</span><span style='float:right'>" + rating.toFixed(2) + "+</span><br>\n"
  }
  return result
}
function format_tag_cloud(title, tags) {
  return "<span class='tag_cloud_title' style='font-weight:bold'>" + title + "</span><br>\n" + format_tags(tags)
}
function show_cluster(cluster) {
  console.log("show_cluster:")
  console.log(cluster)
//   cluster_tags1.text(cluster['tagger'])
//   cluster_tags2.text('Hey!' + cluster['homegrown_tagger'])
//   d3.select(cluster_tags2).text('Hey!')
//   $('<p>HEY!</p>').appendTo('.clusterpanel')
//   $('#clusterpanel').append("<p>hey</p>")
//   $('#clusterpanel').append(format_tags(cluster['homegrown_tagger']))
  cluster_title.text('cluster id: ' + cluster['id'] + ', size: ' + cluster['count'])
  tags1.empty()
  tags1.append(format_tag_cloud('homegrown tagger', cluster['homegrown_tagger']))
  tags2.empty()
  tags2.append(format_tag_cloud('apresta/tagger', cluster['tagger']))
}

var margin = {top: -5, right: -5, bottom: -5, left: -5},
    width = navi_width - margin.left - margin.right,
    height = navi_height - margin.top - margin.bottom;

var zoom = d3.behavior.zoom()
    .scaleExtent([1, 400])
    .on("zoom", zoomed);

// var svg = d3.select("body").append("svg")
var svg = d3.select("#navigator").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.right + ")")
    .call(zoom);

var rect = svg.append("rect")
    .attr("width", width)
    .attr("height", height)
    .style("fill", "none")
    .style("pointer-events", "all");

var scaling_factor = height/70

var container = svg.append("g");


var points = []

var line = d3.svg.line();
line.interpolate("cardinal")
var path = container.append("path")
   .datum(points)
   .attr("class", "line")
   .attr("stroke-dasharray", "2, 2")

var dummyPath = container.append("path")
   .datum(points)
   .attr("class", "noline")
path.call(redraw);

var previousLength = 0
function redraw() {
  dummyPath.attr("d", line)
  totalLength = dummyPath.node().getTotalLength()

  path.attr("stroke-dasharray", totalLength + " " + totalLength)
      .attr("stroke-dashoffset", totalLength - previousLength)
      .transition()
        .attr("d", line)
        .duration(1500)
        .attr("stroke-dashoffset", 0)

  previousLength = totalLength
}

function video_radius(d) {
  if (d['incoming_count']) return d['incoming_count']/ 50 + 1;
  else return 1.0
}

// var color_scale = d3.scale.ordinal()
//         .domain(_.range(0, 170))
//         .range(colorbrewer.RdBu[9]);

var color_scale = d3.scale.category20()

function video_color(d) {
  return color_scale(d['cluster'])
}
function cluster_color(cluster_id) {
  return color_scale(cluster_id)
}

function brighter(string) {
  return d3.rgb(string).brighter()
}
function darker(string) {
  return d3.rgb(string).darker()
}
var selected_video = null
function set_video_selected(d, i) {
  // reset previously selected video
  if (selected_video) {
    selected_video.attr('r', video_radius(selected_video.data()[0]))
//     selected_video.style('fill', 'lightsteelblue')
    selected_video.style('fill', video_color(selected_video.data()[0]))
  }
  // set newly selected video
  parent = d3.select(d3.event.srcElement.parentNode)
  circle = parent.select('circle')
  selected_video = circle
  selected_video.attr('r', video_radius(d)*1.2)
  selected_video.style('fill', brighter(video_color(d)))

  // add this video point to the line-of-visited-videos
  points.push([selected_video.attr('cx'), selected_video.attr('cy')])
  redraw()
}
var selected_cluster = null
function set_cluster_selected(d, i) {
  if (selected_cluster) {
//     selected_cluster.style('fill', cluster_color(selected_cluster.data()[0]['id']))
    selected_cluster.style('stroke', null)
  }
  selected_cluster = d3.select(d3.event.srcElement)
  selected_cluster.style('fill', brighter(cluster_color(d['id'])))
  selected_cluster.style('stroke', 'black')
}

function event_video_clicked(d, i) { show_video(d);
                                     set_video_selected(d, i)}
function event_cluster_clicked(d, i) {show_cluster(d);
                                      set_cluster_selected(d, i)
                                     }

draw_grid_lines(container, width, height, scaling_factor)

function translate_x(x) {return x * scaling_factor + width/2};
function translate_y(y) {return y * scaling_factor + height/2};
function truncate(n, string){
   if (string) {
     if (string.length > n)
        return string.substring(0,n)+'...';
     else
        return string;
   } else {return null}
};

function draw_aura(container, data) {
  aura = container.append("g")
      .attr("class", "aura")
    .selectAll("circle")
      .data(data)
    .enter().append("circle")
      .attr("r", 5.0)
      .attr("cx", function(d) { return translate_x(d['coord'][0]); })
      .attr("cy", function(d) { return translate_y(d['coord'][1]); })
      .attr("opacity", function(d) { if (d['view_count_delta']) return d['view_count_delta']/120000; else return 0; });
//       .attr("opacity", function(d) { if (d['avg_rating_delta']) return d['avg_rating_delta']/0.01; else return 0; });
//       .attr("opacity", function(d) { if (d['num_raters_delta']) return d['num_raters_delta']/3000; else return 0; });
}

function draw(container, cluster, words) {
    container .append("g")
        .attr("transform", "translate("+translate_x(cluster['mean'][0])+","+translate_y(cluster['mean'][1])+")")
      .selectAll("text")
        .data(words)
      .enter().append("text")
        .style("font-size", function(d) { return d.size + "px"; })
        .style("font-family", "Impact")
        .style("fill", function(d, i) { return brighter(cluster_color(cluster['id'])); })
        .style("opacity", 0.7)
        .style("stroke", "black")
        .style("stroke-width", 0.5)
        .attr("text-anchor", "middle")
        .attr("transform", function(d) {
          return "translate(" + [d.x, d.y] + ")rotate(" + d.rotate + ")";
        })
        .text(function(d) { return d.text; });
  }
function create_cloud_layout(container, cluster, tags_key) {
  d3.layout.cloud().size([cluster['sd'][0]*scaling_factor*8+50, cluster['sd'][1]*scaling_factor*8+50])
      .words(
        cluster[tags_key].slice(0,3).map(function(d) {
          return {text: d[0].join(" "),
                  size: 5 + d[1]*0.3,
               };
      }))
      .padding(1)
//       .rotate(function() { return ~~(Math.random() * 2) * 90; })
       .rotate(function() { return 0})
      .font("Impact")
      .fontSize(function(d) { return d.size; })
      .on("end", function(d) {draw(container, cluster, d)})
      .start();
}
function draw_clusters(container, data) {
  if (data==null) {
    console.log("no clusters found in data, skipping")
    set_show_clusters_checked(false)
    return
  }
  set_show_clusters_checked(true)
//   $('.clusterpanel').prop('display', 'block')
//   $('.clusterpanel').css('display', 'block')
//   console.log($('.clusterpanel'))
  d3.select('.clusterpanel').style('display', 'block')
//   $('.clusterpanel').show()


  c = container.append("g").attr("class", "clusters")
  $('.video').insertAfter(c)

  c.selectAll("g")
      .data(data)
    .enter().append("g")
  .append("ellipse")
    .attr("cx", function(d) {return translate_x(d['mean'][0])})
    .attr("cy", function(d) {return translate_y(d['mean'][1])})
    .attr("rx", function(d) {return scaling_factor * 1.5 * d['sd'][0]})
    .attr("ry", function(d) {return scaling_factor * 1.5 * d['sd'][1]})
    .style("fill", function(d) {return darker(cluster_color(d['id']))})
//     .style("stroke", function(d) {return "black"})
    .style("stroke-width", function(d) {return 0.5})
    .on('click', event_cluster_clicked);
}
function draw_tag_clouds(container, data) {
  set_show_tag_clouds_checked(true)
  c = container.append("g").attr("class", "tag_clouds")
  for (i=0; i!= data.length; i++) {
//     create_cloud_layout(c, data[i], 'tagger')
    create_cloud_layout(c, data[i], 'homegrown_tagger')
  }
}
function undraw_clusters(container) {
  container.selectAll("g g.clusters").remove()
}
function undraw_tag_clouds(container) {
  container.selectAll("g g.tag_clouds").remove()
}
function undraw_videos(container) {
  container.selectAll("g g.video").remove()
}
function draw_videos(container, data) {
  set_show_videos_checked(true)

  c = container.append("g")
      .attr("class", "video")
    .selectAll("g")
      .data(data)
    .enter().append("g");
  c.append("circle")
      .attr("r",  function(d) { return video_radius(d)})
      .attr("cx", function(d) { return translate_x(d['coord'][0]); })
      .attr("cy", function(d) { return translate_y(d['coord'][1]); })
      .style("fill", function(d) { return cluster_color(d['cluster'])})
  c.append("text")
      .attr("x", function(d) { return translate_x(d['coord'][0]) + 1.5; })
      .attr("y", function(d) { return translate_y(d['coord'][1]) + 0.5; })
      .text( function (d) { return truncate(12, d['video.title'])})
      .attr("font-size", "2px");
  c.on('click', event_video_clicked);
}

file_path = getQueryVariable("data")
if (!file_path) file_path = "encoding.json"
//window.alert(file_path)
file_path = '/data/' + file_path
d3.json(file_path, function(error, data) {
  // data includes 'data', 'clusters'

//   draw_aura(container, dots)
  set_show_clusters_functions(
    function() {draw_clusters(container, data['clusters'])},
    function() {undraw_clusters(container)})

  set_show_tag_clouds_functions(
    function() {draw_tag_clouds(container, data['clusters'])},
    function() {undraw_tag_clouds(container)})

  set_show_videos_functions (
    function() {draw_videos(container, data['data'])},
    function() {undraw_videos(container)})

  draw_videos(container, data['data'])
//  draw_clusters(container, data['clusters'])
});

// thumbnail: http://i.ytimg.com/vi/<video-id>/0.jpg

function zoomed() {
  container.attr("transform", "translate(" + d3.event.translate + ")scale(" + d3.event.scale + ")");
  // d3.event.translate is the vector to translate each data point center,
  // dependent on scale and position
}

</script>

</body>

