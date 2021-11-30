# devops-netologyFirst line
игнорирруются 
1) все файлы в поддиректория /.terraform
2) содержащие .tfstate
3) crash.log рекурсивно
4) оканчивающиеся на .tfvars
5) оканчивающиеся _override.tf или _override.tf.json + override.tf или override.tf.json рекурсивно
6) .terraformrc и terraform.rc рекурсивно
