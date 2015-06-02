# HTML-DOM-Tree

package structures;

import java.util.*;

/**
 * This class implements an HTML DOM Tree. Each node of the tree is a TagNode, with fields for
 * tag/text, first child and sibling.
 * 
 */
public class Tree {
	
	
	/**
	 * Root node
	 */
	TagNode root=null;
	
	/**
	 * Scanner used to read input HTML file when building the tree
	 */
	Scanner sc;
	
	/**
	 * Initializes this tree object with scanner for input HTML file
	 * 
	 * @param sc Scanner for input HTML file
	 */
	public Tree(Scanner sc) {
		this.sc = sc;
		root = null;
	}
	
	/**
	 * Builds the DOM tree from input HTML file. The root of the 
	 * tree is stored in the root field.
	 */
	public void build() {
		
		root = builder();
	}
	
	private TagNode builder(){
		boolean isTag = false;//verifies that the line is a tag
		String line = null;//variable to hold next line in sc
		
		if(sc.hasNext())//checks to see if there is a next line in the scanner
			line = sc.nextLine();//stores next line in the scanner
		else
			return null;//if there are no more lines, then return null
		
		if(line.length() > 1 && line.charAt(1) == '/')
			return null;//if it is an ending tag (i.e.</em>) then return null
		
		if(line.charAt(0) == '<'){
			line = line.substring(1, line.length()-1);
			isTag = true;//if the next line is a tag then you store it in the line variable
		}
		
		TagNode t = new TagNode(line, null, null);//store the line in the TagNode variable
		
		if(isTag)
			t.firstChild = builder();//if it is a tag, then you go onto the next line and append it as a firstChild of the tag
		
		t.sibling = builder();//if it is not tag, then it cannot have a first child so it is plain text so you have a sibling for it
		
		return t;

	}
	
	/**
	 * Replaces all occurrences of an old tag in the DOM tree with a new tag
	 * 
	 * @param oldTag Old tag
	 * @param newTag Replacement tag
	 */
	public void replaceTag(String oldTag, String newTag) {
		replaceTag(root, oldTag, newTag);//recursive call
	}
	
	private static void replaceTag(TagNode root, String oldTag, String newTag) {
		
		TagNode current = root;//temp variable assignment
		
		
		if (current == null)//base case
			return;
		
		
		if ((oldTag.equals("p") || oldTag.equals("em") || oldTag.equals("b")) && (newTag.equals("p") || newTag.equals("em") || newTag.equals("b")) && current.tag.equals(oldTag)) {
			current.tag = newTag;//has to meet these requirements to change if p em or b
		}
		else if ((oldTag.equals("ol") || oldTag.equals("ul")) && (newTag.equals("ol") || newTag.equals("ul")) && current.tag.equals(oldTag)) {
			current.tag = newTag;//has to meet these requirements to change if ol or ul
		}
		
		
		
		replaceTag(current.firstChild, oldTag, newTag);
		replaceTag(current.sibling, oldTag, newTag);
	}
		
	

	
		
	
	
	/**
	 * Boldfaces every column of the given row of the table in the DOM tree. The boldface (b)
	 * tag appears directly under the td tag of every column of this row.
	 * 
	 * @param row Row to bold, first row is numbered 1 (not 0).
	 */
	public void boldRow(int row) {
		boldRow(row, root);
	}

	
	private void boldRow(int row, TagNode root){
		
		
		int count=1;
		
		
		Stack<TagNode> s = new Stack<TagNode>();
		   if(root==null)
			   return;
		   TagNode current=root;
		   
		   while(!s.isEmpty() || current != null){
			   if(current != null){
				   if(current.tag.equals("table")){
						TagNode tmp=current.firstChild;
						while(count != row){
							tmp=tmp.sibling;
							count++;
						}
						TagNode below=tmp.firstChild;
						TagNode boldTag=new TagNode("b", below.firstChild, null);
						below.firstChild=boldTag;
						boldColumns(below);
					}
				   s.push(current);
				   current=current.firstChild;
			   }
			   else{
				   current=s.pop();
				   current=current.sibling;
			   }
		   
		
		   }	
		
		
		
		
	}
	private static void boldColumns(TagNode x){//bolds all of the columns for a given row so the whole row is bold
		if(x.sibling != null){
			x=x.sibling;
			TagNode bolden=new TagNode("b", x.firstChild, null);
			x.firstChild=bolden;
			boldColumns(x);
			
		}
		else
			return;
		
	} 
	
	
	
	
	/**
	 * Remove all occurrences of a tag from the DOM tree. If the tag is p, em, or b, all occurrences of the tag
	 * are removed. If the tag is ol or ul, then All occurrences of such a tag are removed from the tree, and, 
	 * in addition, all the li tags immediately under the removed tag are converted to p tags. 
	 * 
	 * @param tag Tag to be removed, can be p, em, b, ol, or ul
	 */
	public void removeTag(String tag) {
		if (root == null)//base case
			return;
		else 
			if(!this.hasTag(tag, root))//if the tag does not exist then this returns null
				return;
			else{
				removeTag(tag, root, root.firstChild);//if tag exists then the program runs
				removeTag(tag);//gets rid of tag and the program runs again
			}
	}

	private static void removeTag(String tag, TagNode prev, TagNode current) {
		if (current == null)//base case
			return;
		else if(prev==null)//base case
			return;
		else if (current.tag.equals(tag)){//checks to see if the current node's data equals tag

			if (tag.equals("ol")){//spots ol tag
				TagNode ol=current;
				while(ol != null){//traverses through siblings
				if(current.tag.equals("li"))
					current.tag="p";//changes any occurrence of li to p
				ol=ol.sibling;
				}
			}
			if(tag.equals("ul"))//spots ul tag
			{
				TagNode ul=current;
				while(ul != null){//traverses through siblings
				if(current.tag.equals("li"))
					current.tag="p";//changes any occurrence of li to p
				ul=ul.sibling;
				}
			}

			if (prev.firstChild == current) {//Case one where current is a child of prev
				prev.firstChild = current.firstChild;
				TagNode end1=current.firstChild;
				while(end1.sibling != null){
					end1=end1.sibling;
				}
				end1.sibling=current.sibling;//if a tag has siblings then the tag gets removed but the siblings remain
				
			} else if (prev.sibling == current) {//Case two where current is a sibling of prev
				TagNode end2=current.firstChild;
				while(end2.sibling != null){
					end2=end2.sibling;
				}
				end2.sibling=current.sibling;
				
				prev.sibling = current.firstChild;//if there a string of siblings underneath the tag, then that gets copied into the sequence and the tag is deleted
			}

			return;
		}

		
		removeTag(tag, current, current.firstChild);
		removeTag(tag, current, current.sibling);//in order traversal
		
		
	}




	private static boolean hasTag(String tag, TagNode current) {
		if (current == null)//base case for recursive call
			return false;
		else if (current.tag.equals(tag))//checks if tag exists in the tree
			return true;
		boolean children=hasTag(tag, current.firstChild);//tests out all of the children of the tree
		boolean siblings=hasTag(tag, current.sibling);//tests out all of the siblings of the tree

		return children || siblings;//returns true if either one of them is true
	}
	
	/**
	 * Adds a tag around all occurrences of a word in the DOM tree.
	 * 
	 * @param word Word around which tag is to be added
	 * @param tag Tag to be added
	 */
	public void addTag(String word, String tag) {
		
	if(tag.equals("b") || tag.equals("em"))	//only works for p and em
		addTag2(null, root, word, tag);
	else
		return;//returns otherwise
	}
	
	private void addTag2(TagNode prev, TagNode current, String word, String tag){
		word=word.toLowerCase();
		
		if(current==null)//if node is null then return
			return;
		
		if(prev != null && prev.tag.equals(tag) && !current.tag.toLowerCase().contains(word))//checks to make sure that there are no embedded tags
			return;
		
		
			if(current.tag.toLowerCase().contains(word)){//checks if tag contains the word
				
				
				String[] breakup=current.tag.split(" ");//divides the tag if it has multiple words
				int count=0;
				for(int i=0; i<breakup.length; i++){
					if(isEqual(breakup[i], word)){
						count++;
						
					}
				}//counts number of occurrences
				
			
				
				if(breakup.length==1){//case if it is only the word
					if(isEqual(breakup[0], word)){
						if(prev.firstChild==current && current.sibling==null){
							TagNode t=new TagNode(tag, current, null);
							prev.firstChild=t;
						}
						else if(prev.firstChild==current && current.sibling != null){
							TagNode t=new TagNode(tag, current, current.sibling);
							prev.firstChild=t;
							current.sibling=null;
						}
				}
				}
				else if(breakup.length==2){//case for if there are two words in the tag
					int s=0;
					
					TagNode tNode=new TagNode(null, null, null);
				
					
					
					
					if(isEqual(breakup[0], word)){//if the word is equal to the tag then this returns true
						String target=breakup[0];
						if(prev.firstChild==current){
						tNode.tag=target;
						
						TagNode after=new TagNode(breakup[1], null, null);
						TagNode tags=new TagNode(tag, tNode, after);
						prev.firstChild=tags;
						after.sibling=current.sibling;
						tNode.sibling=null;
						addTag2(tags, tags.sibling, word, tag);
						}
						else{
							tNode.tag=target;
							TagNode after=new TagNode(breakup[1], null, null);
							TagNode tags=new TagNode(tag, tNode, after);
							prev.sibling=tags;
							after.sibling=current.sibling;
						}
					}
					else{
						String target=breakup[1];
						if(prev.firstChild==current){
						tNode.tag=target;
						TagNode before=new TagNode(breakup[0], null, null);
						prev.firstChild=before;
						TagNode tags=new TagNode(tag, tNode, null);
						before.sibling=tags;
						tags.sibling=current.sibling;
						tNode.sibling=null;
						}
						else{
						tNode.tag=target;
						TagNode before=new TagNode(breakup[0], null, null);
						prev.sibling=before;
						TagNode tags=new TagNode(tag, tNode, null);
						before.sibling=tags;
						tags.sibling=current.sibling;
						tNode.sibling=null;
						}
						
					}
					
					
				}
				
				else if(breakup.length>2 && prev.firstChild==current){//if the number of words is greater than 2 and it is a firstChild
				String before="";
				String target="";
				String after="";
				int match=0;
				TagNode beforeNode=new TagNode(null, null, null);
				TagNode targetNode=new TagNode(null, null, null);
				TagNode afterNode=new TagNode(null, null, null);
				for(int i=0; i<breakup.length; i++){
					if(isEqual(breakup[i], word)){
						System.out.println(breakup[i]);
						match=i;
						target=breakup[i];
						targetNode.tag=target;
						break;
					}
				}
				for(int i=0; i<breakup.length; i++){
					if(i<match){
						before+=breakup[i] + " ";
					}
					if(i>match)
						after+=breakup[i] + " ";
				}
				beforeNode.tag=before;
				afterNode.tag=after;
				
				if(isEqual(breakup[0], word)){//check if the word is at the beginning of the tag
					TagNode tags=new TagNode(tag, targetNode, afterNode);
					prev.firstChild=tags;
					afterNode.sibling=current.sibling;
					targetNode.sibling=null;
					
					addTag2(tags, tags.sibling, word, tag);
				}
				else if(isEqual(breakup[breakup.length-1], word) && count==1){
					TagNode tags=new TagNode(tag, targetNode, current.sibling);
					beforeNode.sibling=tags;
					prev.firstChild=beforeNode;
					targetNode.sibling=null;
				}
				
				else{//default case
				
				TagNode tags=new TagNode(tag, targetNode, afterNode);
				beforeNode.sibling=tags;
				prev.firstChild=beforeNode;
				targetNode.sibling=null;
				afterNode.sibling=current.sibling;
				addTag2(tags, tags.sibling, word, tag);
				}
				}
				else if(breakup.length>2 && prev.sibling==current){//checks if the number of words in the tag is greater than 2 and checks if prev.sibling=current
					
					String before="";
					String target="";
					String after="";
					int match=0;
					TagNode beforeNode=new TagNode(null, null, null);
					TagNode targetNode=new TagNode(null, null, null);
					TagNode afterNode=new TagNode(null, null, null);
					for(int i=0; i<breakup.length; i++){
						if(isEqual(breakup[i], word)){
							
							match=i;
							target=breakup[i];
							targetNode.tag=target;
							break;
						}
					}
					for(int i=0; i<breakup.length; i++){
						if(i<match){
							before+=breakup[i] + " ";
						}
						if(i>match)
							after+=breakup[i] + " ";
					}
					beforeNode.tag=before;
					afterNode.tag=after;
					if(isEqual(breakup[0], word)){//at the beginning case
						TagNode tags=new TagNode(tag, targetNode, afterNode);
						afterNode.sibling=current.sibling;
						prev.sibling=tags;
						targetNode.sibling=null;
						addTag2(tags, tags.sibling, word, tag);
					}
					else if(isEqual(breakup[breakup.length-1], word) && count==1){
						TagNode tags=new TagNode(tag, targetNode, current.sibling);
						beforeNode.sibling=tags;
						prev.sibling=beforeNode;
						targetNode.sibling=null;
					}
					
					
					else{
						TagNode tags=new TagNode(tag, targetNode, afterNode);
						prev.sibling=beforeNode;
						beforeNode.sibling=tags;
						afterNode.sibling=current.sibling;
						targetNode.sibling=null;
						if(count >= 2)
							addTag2(tags, tags.sibling, word, tag);
						
					}
						
				}
			
			
			
				
			}		
			
			addTag2(current, current.firstChild, word, tag);
			addTag2(current, current.sibling, word, tag);
	
	
	}
	
	

	
	private static boolean isEqual(String word, String target){
		word = word.toLowerCase();
		target = target.toLowerCase();
		Character punct=word.charAt(word.length()-1);
		if(punct=='?' || punct=='!' || punct==':' || punct==';' || punct=='.')
			word=word.replace(word.substring(word.length()-1, word.length()-1), "");
		if(target.equals(word))
			return true;
		if(target.equals(word.substring(0, word.length()-1)))
			return true;
		else
			return false;
	}
	
		
	

	/**
	 * Gets the HTML represented by this DOM tree. The returned string includes
	 * new lines, so that when it is printed, it will be identical to the
	 * input file from which the DOM tree was built.
	 * 
	 * @return HTML string, including new lines. 
	 */
	public String getHTML() {
		StringBuilder sb = new StringBuilder();
		getHTML(root, sb);
		return sb.toString();
	}
	
	private void getHTML(TagNode root, StringBuilder sb) {
		for (TagNode ptr=root; ptr != null;ptr=ptr.sibling) {
			if (ptr.firstChild == null) {
				sb.append(ptr.tag);
				sb.append("\n");
			} else {
				sb.append("<");
				sb.append(ptr.tag);
				sb.append(">\n");
				getHTML(ptr.firstChild, sb);
				sb.append("</");
				sb.append(ptr.tag);
				sb.append(">\n");	
			}
		}
	}
	
}
