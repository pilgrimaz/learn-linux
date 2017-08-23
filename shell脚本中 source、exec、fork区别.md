#### shell脚本中 source、exec、fork区别							

1. source、fork和exec都是在一个脚本中调用第二个脚本(B.sh)
2. source、fork执行完第二个脚本(B.sh)后，会继续往下执行父脚本(A.sh)，而exec执行完第二个脚本(B.sh)不再继续执行父脚本(A.sh)
3. source的时候，会把第二个脚本(B.sh)里面的变量带出来给父脚本(A.sh)，fork并不会把第二个脚本中(B.sh)的变量带出来给父脚本(A.sh)


