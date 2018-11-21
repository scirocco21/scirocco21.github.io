---
layout: post
title:      "DoubleCheck - my final project, built with React/Redux"
date:       2018-11-21 21:07:05 +0000
permalink:  doublecheck_-_my_final_project_built_with_react_redux
---

Hoooray! My final portfolio has finally come together to a point where it's worth submitting. The idea behind it is let users paste in shorter snippets of texts, e.g. tweets and the like, which will get posted to Watson's Tone Analyzer API behind the scenes. The App then displays the results of the analysis to the user and gives them the option of saving it as a project. The Rails backend handles data persistence, while all the routing is done via React Router. 

Below I go through a few of the technical challenges posed by the project, in no particular order:

## Dispatching asynchronous actions with Thunk

Figuring out how to Integrate API requests into the React/Redux flow was a steep learning curve. For example, to delete a project, a user would send a delete request to the Rails server. Only when the server responds with some data that the deletion was successful should the Redux store (which holds a copy of the projects in its state) be updated. Otherwise, there is risk of the server and the global state of the app being out of sync, which could quickly turn into a big mess. The tricky part is that there is prior way of telling how long the delete request will take to complete, so one can't simply chain one action to another and expect them to be executed correctly one after the other. That's where Thunk can come in handy: 

```

export function deleteData(project) {
  return (dispatch) => {
    return fetch(`/api/projects/${project.id}`, {
      method: 'delete'
    })
  .then(response => response.json())
  .then(data => dispatch({type: "DELETE_PROJECT", payload: data.project.id}))
  }
}
```

Unlike the normal React/Redux flow, where `dispatch`, well, dispatches a Javascript object (or an action) to the store, Thunk returns a function, which in turn returns the relevant API call to the server. By adding a promise chain, we can thus ensure that the delete action is only dispatched to the store once the API call has actually returned data and the promises have succesfully been resolved. 

## Client-side routing with react-router-dom

Routing in React can present its own unique set of challenges compared to server-side routing in Rails, for example. One peculiar feature of `react-router` is that URL parameters (as one would find when requesting e.g. `/projects/:id`) are passed down to a component through the props object, which contains keys such as `history`, from where params can be extracted. 

There's a catch, though: if a container component is rendered via a Route, and that container component contains a child that needs to have access to `params`, the child component needs to be wrapped in `withRouter` when exporting it. 

To give a more concrete example, my App has Router for creating new projects (called '/projects/new'), which renders a Home Component with a set of props:
```

  <Route exact path='/projects/new' render={(props) => (<div><Home {...props} buttonVisible={false}/></div>)}/>
```

Inside the Home Component, I would now have access to e.g. `this.props.history.params`. But `Home` itself contains a `TextForm` which needs to have access to `params`, even though it is not directly rendered by a Route:
```

import {withRouter} from 'react-router-dom'

class TextForm extends Component {
  
}
```

```

export default withRouter(connect(mapStateToProps, mapDispatchToProps)(TextForm))
```


## Processing complex API data

Another challenge with my final project has been handling complex API data and modelling it appropriately on the server and client side. Here is an example of what Watson's Tone Analyzer returns:
```
{
  "document_tone":
	  {"tones":[
		  {"score":0.670944,"tone_id":"sadness","tone_name":"Sadness"},    
			{"score":0.71572,"tone_id":"tentative","tone_name":"Tentative"}
			]
		},
	"sentences_tone":[
	   {"sentence_id":0,"text":"extremely disappointed with the level of service with Virgin Media.",
		 "tones":[
		    {"score":0.64031,"tone_id":"sadness","tone_name":"Sadness"},  
			  {"score":0.618451,"tone_id":"confident","tone_name":"Confident"}
			]
		},
		{"sentence_id":1,"text":"Have been trying to get a new landline installed almost 4 weeks and no landline but being   
		billed for it.","tones":[
		   {"score":0.704683,"tone_id":"tentative","tone_name":"Tentative"}
			 ]
			},
		{"sentence_id":2,"text":"Poor service and staff don’t know what they are doing.","tones":
		[
		  {"score":0.687768,"tone_id":"analytical","tone_name":"Analytical"}]
		}
	]
}


```

Upon closer inspection, it's clear that `tones` are shared by both the document as a whole, and by each sentence individually. When modelling this kind of relationship between models in Rails, there is a handy trick that lets two model objects have many instances of the same resource ( a so-called *polymorphic association*):

```
class Project < ApplicationRecord
  has_many :tones, as: :analyzable
  has_many :sentences
  accepts_nested_attributes_for :tones, :sentences
end

class Sentence < ApplicationRecord
  belongs_to :project
  has_many :tones, as: :analyzable
  accepts_nested_attributes_for :tones
end

class Tone < ApplicationRecord
  belongs_to :analyzable, polymorphic: true
end
```

Thus, instead of modelling e.g. `ProjectTone` and `SentenceTone` separately, both `Project` and `Sentence` models have access to `Tone` through the intermediary of `analyzable`, which prevents unnecessary code duplication. 



