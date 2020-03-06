# ePubJS---search-text
Text search routine for the ePubJS project. Searches text (case insensitive) on per text-node basis.
**_It uses the 'rangy' plugin, installed via npm._**

code:

```
searchAction(search_text, hog_tie){// 'hog_tie' toggles between 'SearchChapter' and 'SearchBook'
    
  this.searchResult = [];
      
  return new Promise((resolve,reject) =>{

    this.ePub.spine.each((item) => {
      item.load( this.ePub.book.load.bind(this.ePub.book) ).then((contents) => {
        
        let bod = contents.getElementsByTagName('body')[0];
  
        var rangey = rangy.createRange();
        rangey.selectNode(bod);
  
        let textNodes = rangey.getNodes([3], (node) => {
          let foo = search_text; //'produce';
          let regExp = new RegExp("\\b" + foo + "\\b", "i"); // the word 'foo'
          return regExp.test(node.data);
        });//the above will obtain all the text nodes within the document body that contain the word in the variable "foo", case insensitively:
        
        var found: any = [], itemx: any = {}, itemers: any = [];
        textNodes.forEach(textNode => {
          found.push(textNode);
        });
        itemx[item.href] = found;
        itemers.push(item.href);
        for (let itx = 0; itx < itemers.length; itx++) {
          const itemer = itemers[itx];
          
          if(!((itemer !== this.ePub.currentHref) && (hog_tie === 'SearchChapter'))){
            if(itemx[itemer].length > 0){

              /* what matters */

              // create search value range..
              var reSrange = rangy.createRange();
              (itemx[itemer]).forEach(foundx => {

                var iot: number = 0;
                while( foundx.textContent.toLowerCase().indexOf( (search_text).toLowerCase(), iot ) > -1 ){

                  reSrange.setStart(foundx, foundx.textContent.toLowerCase().indexOf( (search_text).toLowerCase(), iot ) )
                  reSrange.setEnd(foundx, foundx.textContent.toLowerCase().indexOf( (search_text).toLowerCase(), iot ) + (search_text).length )

                  let mcontents = this.ePub.movingContentOBJECT[itemer];//this.ePub.currentHref
                  var searchCFI = mcontents.cfiFromRange( reSrange );

                  this.searchResult.push({
                    href: itemer,//this.ePub.currentHref
                    cfi: searchCFI.toString(),
                    range: reSrange
                  })

                  iot = reSrange.endOffset;
                }//while

              });
              
            }
          }//hog_tie
        }//itemers
        
        return resolve(this.searchResult);
  
      });
    });
  });
}
```
