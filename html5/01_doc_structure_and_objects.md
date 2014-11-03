#Implement and Manipulate Document Structures and Objects
##Create Sturcture

- Html 5 has new tags and some have been deprecated from html4.
- 30 new, 12 tags deprecated
- New 
-- article (block) independent block of content, should be standalone, eg a blog post. there may be sub-articles and they usually contain sections.
Should be independently redistributable such as in syndication.
-- aside 
-- audio
-- bdi
-- canvas
-- command
-- datalist
-- details
-- embed
-- figure semantic but not structural. denotes standalone illustrative content.
-- figcaption caption of the above
-- footer opposite of header. attributions, copyright, etc.
-- header contains heading content, often nav. site or page level. contains hgroup, h* elements, sometemes search, logo etc.
-- hgroup a group of headings. should contain only h-tags. hint to html outlining engine to only contain the first H element in the outline.
-- keygen
-- mark a section of highlighted text referred to elsewhere.
-- meter
-- nav (block) navigational feature, usually site-level. The section of the page which links to other pages or parts of the page. Not all links need 
to be in a nav. Eg where you link to ToS in a footer - doesn't ahve to be a nav. Major navigation only.
-- output
-- progress
-- rp
-- rt
-- ruby
-- section (block) grouping of content. typically includes a heading. thematic grouping of content. appears inside aricle
-- source
-- summary
-- time a time or date representation.
-- track
-- video
-- wbr

- Some of these tags will "section" content - which aids in outlining. IE If you have a h2 outside and article, it will be treated like the parent of an h1 inside the article.
- the goal of these is to express meaning in the content and not css class or ids. this improves standardization of structrue (eg .arcticle vs .feature) and aits in accesibility.
- signal html5 content with the DOCTYPE <!doctype html>. also include a charset meta tag. 
- support
-- IE 9+, Firefox 16+ Chrome 23+ Safari 5.1 and iOS Safari 4.0, android 2.2 and Blackberry. Use shims or javascript for back compatibility.

##Layout in HTML
- Define the elements that provide the overall structire to a page. Head/footer positionging, nave, sidebar, content etc
- Main css attribute is position
-- position:absolute offset taken from the first parent with a non-static position. do not participate in normal layout flow
-- position:relative can be offset from its original/inherited position, moves but originally reserved space is still preserved
-- position:static flowed normally, can't set dist to trbl
-- position:fixed relative to browser window and doesn't move when scrolled.

- z-index affects overlapping elements. order on the screen, lower goes behind, equal means last item in doc goes on top
- most modern layouts use float. 
-- allows sibling content to flow around an element if there is foom, otherwise it wraps floated elements should come before float:none; elements.
