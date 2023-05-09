# Register new user 
kubectl exec --namespace wg -it hs-9f97f7c69-s6fvs -- headscale users create <USERNAME>

# Request auth 
tailscale up --login-server https://hs.regacloud.ch

# Register new client
kubectl exec --namespace wg -it hs-9f97f7c69-7cdpb -- headscale nodes register --user backend-eb-i --key nodekey:2f3d3c1ae8e11fa1c870b00c8e409fc9498a26b407fc810fe98942b57e78e501



Z0dsKCeDBA.2h79W0e8iqCtaDk22p7LB3FnYBC9n8Z-ez6wwpjnE1g