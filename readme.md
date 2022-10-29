# Run Digitalocean Droplet As Runner

## Example usage

```
name: do-the-job
on:
  workflow_dispatch:
jobs:
  start-do-runner:
    name: start self-hosted digitalocean runner
    runs-on: ubuntu-latest
    outputs:
      label: ${{ steps.start-digitalocean-runner.outputs.label }}
      do-droplet-id: ${{ steps.start-digitalocean-runner.outputs.do-droplet-id }}
    steps:
      - name: start digitalocean runner
        id: start-digitalocean-runner
        uses: batpool/digitalocean-github-runner@v1.1
        with:
          mode: start
          github-token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
          do-pat: ${{ secrets.DO_PERSONAL_ACCESS_TOKEN }}
          do-region: blr1
          do-image-id: ubuntu-22-10-x64
          do-droplet-type: s-1vcpu-1gb
  
  do-the-job:
    name: do the job on the runner
    needs: start-do-runner 
    runs-on: ${{ needs.start-do-runner.outputs.label }}
    steps:
      - name: test
        run: |
          cat /etc/os-release
  
  stop-do-runner:
    name: stop self-hosted digitalocean runner
    needs:
      - start-do-runner
      - do-the-job
    runs-on: ubuntu-latest
    if: ${{ always() }}
    steps:
      - name: stop digitalocean runner
        uses: batpool/digitalocean-github-runner@v1.1
        with:
          mode: stop
          github-token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN }}
          do-pat: ${{ secrets.DO_PERSONAL_ACCESS_TOKEN }}
          label: ${{ needs.start-do-runner.outputs.label }}
          do-droplet-id: ${{ needs.start-do-runner.outputs.do-droplet-id }}
```