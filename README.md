# is-tag-reachable-from-default-branch

This action takes in a tag and determines if it is reachable from the default branch.  This action should be used in conjunction with the `actions/checkout` action where a fetch-depth has been set so the action has access to tags and history.

When this executes it will:
1. Checkout the default branch if it is not currently on it (the `ref` arg should be provided in this case)
2. Retrieve the list of tags that are reachable from the default branch 
3. Check if the tag is in that list of reachable tags
4. If the tag is not in the reachable list it will set the Exit Code to 1 unless the `error-if-not-reachable` input is set to false.  
5. Switch back to the original branch if a `ref` arg was provided

If you are working on something other than the default branch, it may be best to run this earlier in your workflow so switching branches doesn't cause loss of changes.
    

## Inputs
| Parameter                | Is Required | Description                                                                                                                                                                                                    |
| ------------------------ | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tag`                    | true        | The tag to check if it is reachable from the default branch.                                                                                                                                                   |
| `default-branch`         | false       | The default branch of the repository.  Defaults to main.                                                                                                                                                       |
| `ref`                    | false       | Required when a `ref` arg was used with `actions/checkout` or if a non-default branch was selected when starting the workflow.  Specify the same ref here so the action can return the repository to that ref. |
| `error-if-not-reachable` | false       | Determines whether the action should throw an error if the tag is not reachable on the default branch.  Defaults to true.                                                                                      |


## Outputs
| Output      | Description                                                 |
| ----------- | ----------------------------------------------------------- |
| `reachable` | true or false value indicating whether the tag is reachable |

## Example

```yml
jobs:
  # Assumes the default branch is main and that the workflow is being run from the main branch
  validate-tag-to-deploy-default-branch:
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v2
        
      - name: Check if tag is reachable by main
        uses: im-open/is-tag-reachable-from-default-branch@v1.0.0
        with:
          tag: 'latest'
  
  # Assumes the default branch is main and that the workflow is being run from a non-default branch
  validate-tag-to-deploy-from-non-default-workflow-branch:
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v2
        
      - name: Check if tag is reachable by main
        uses: im-open/is-tag-reachable-from-default-branch@v1.0.0
        with:
          tag: 'latest'
          ref: ${{ github.ref }}

  # Specifies the default branch is master, errors are handled outside the workflow and that
  # the action should return to v2.0.12 once it has finished checking
  validate-tag-to-deploy-advanced:
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v2
        with: 
          ref: 'v2.0.12'
        
      - name: Check if tag is reachable by master
        id: tag-check
        uses: im-open/is-tag-reachable-from-default-branch@v1.0.0
        with:
          tag: 'latest'                 # The tag to check
          error-if-not-reachable: false # Don't throw an error if the tag is not reachable
          default-branch: master        # The default branch is master in this case, not main
          ref: 'v2.0.12'                # This is the branch/tag/sha that has been checked out
      
      - name: Exit if tag is not reachable
        if: ${{ steps.tag-check.outputs.reachable }} == false
        run: |
          echo "The tag ${{ github.events.inputs.tag }} is not reachable on the default branch"
          exit 1


  deploy-to-prod:
    needs: [validate-tag-to-deploy-advanced]
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v2
        with: 
          ref: 'v2.0.12'
        
      - name: Prod Deploy
        run: ./deploy-to-prod
```


## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2021, Extend Health, LLC. Code released under the [MIT license](LICENSE).
