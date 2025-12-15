# wtaaf
what the! another agent framework?

agent frameworks are too extra this one is simpler i hope

ideally, easy to add functionality dynamically (not too much code required to do anything beyond min required JS). easy to hook	into lifecycle to introduce edge case handling and dynamic observability

## food for thought

since the state is raw js objects, you can do all the magic you want with it.  
- e.g., if you're using it on the frontend, pass in a vue `reactive` and then you have global state and reactivity across the whole frontend! (actually you could do this anywhere with `@vue/reactivity`)
- since everything except the builder functions is raw js, add human in the loop and other things very trivially using your own state!

## spec 

stateful agent loops

```typescript
import { agent, toolset, run } from "wtaf";

const researchAgent = agent("gemini-3.0-preview", /*...*/);

// very declarative and expressive syntax to build an agent
const codingAgent = agent("gpt-5.2", () => ({
	goalDefined: false,
	goalCompleted: false,
	goal: null as string | null,
	fileSummaries: {} as Partial<Record<string, string>>,
}))
	.tool`requestClarification`(RequestClarificationSchema, (props) => requestClarification(props.reason))
	.tool`readFileSummary`(ReadFileSummarySchema, (props, state) => state.fileSummaries[props.filePath])
	.tool`readCompleteFile`(ReadFileSchema, (props) => readFile(props.filePath))
	.tool`webSearch`(WebSearchSchema), webSearch)
	.guard(state => !state.goalDefined, toolset()
		.tool`defineGoal`(DefineGoalSchema, (props, state) => {
			state.goalDefined = true;
			state.goal = props.goal;
		})
	])
	.guard(state => state.goalDefined, toolset()
		.tool`listFiles`(ListFilesSchema, (props) => listFiles(props.rootDir))
		.tool`createFile`(CreateFileSchema, (props, state) => {
			 createFile(props.filePath, props.content);
			 state.fileSummaries[props.filePath] = props.fileSummary;
		})
		.tool`editFile`(EditFileSchema, (props, state) => {
			editFile(props.filePath, props.diff);
			state.fileSummaries[props.filePath] = props.adjustedFileSummary;
		}),
	])
	.stop(state => state.goalCompleted);


// configurable observability!
run(codingAgent)
	.onIterationCompleted((state, ctx) => { /*ctx.stop()*/ })
	.onToolCalled((state, ctx) => {})
	.onToolCallCompleted((state, ctx) => {})
	/* ... */
	.begin();
```
