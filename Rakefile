require 'rubygems'
require 'bundler/setup'
require 'json'
require 'open-uri'
require 'aws-sdk-core'
require 'set'

SERVICE       = 'aptbot'
TASK_FAMILY   = 'aptbot'
CLUSTER       = ENV['CLUSTER'] || 'default'
VERSION       = ENV['CIRCLE_BUILD_NUM'] || 0
DESIRED_COUNT = 1

TASK_TEMPLATE = "#{SERVICE}-task.json"
TASK_FILE     = "#{SERVICE}-task-v#{VERSION}.json"

# use ENV['AWS_ACCESS_KEY_ID'] and ENV['AWS_SECRET_ACCESS_KEY'] if you don't want to set credentials by code
def credentials=(credentials)
  unless credentials == @credentials
    @lazy_cloud_formation = nil
    @credentials = credentials
  end
end

def region=(region) # use ENV['AWS_REGION'] or ENV['AWS_DEFAULT_REGION']
  unless region == @region
    @lazy_cloud_formation = nil
    @region = region
  end
end

def ecs # lazy ECS client
  unless @ecs
    params = {}
    params[:credentials] = @credentials if @credentials
    params[:region] = @region if @region
    @lazy_ecs = Aws::ECS::Client.new(params)
  end
  @lazy_ecs
end

def find_service(cluster, service)
  status = ecs.describe_services(cluster: cluster, services: [service]).services.first.status
  puts "SERVICE: " + status
  status == "ACTIVE"
end


desc "create or update service #{SERVICE}"
task :create_or_update do

  # Create a new task definition for this build
  taskDefintion = File.read(TASK_TEMPLATE)
  taskDefintion = taskDefintion.sub(/BUILD_NR/, VERSION)

  result = ecs.register_task_definition(family: TASK_FAMILY, container_definitions: JSON.parse(taskDefintion))
  fullQualifiedTaskDefinition = "#{TASK_FAMILY}:#{result.task_definition.revision}"

  if find_service(CLUSTER, SERVICE)
    puts "UPDATE SERVICE #{SERVICE} on cluster #{CLUSTER} using task definition #{fullQualifiedTaskDefinition}"
    result = ecs.update_service(cluster: CLUSTER, service: SERVICE, task_definition: fullQualifiedTaskDefinition, desired_count: DESIRED_COUNT)
  else
    puts "CREATE SERVICE #{SERVICE} on cluster #{CLUSTER} using task definition #{fullQualifiedTaskDefinition}"
    ecs.create_service(cluster: CLUSTER, service_name: SERVICE, task_definition: fullQualifiedTaskDefinition, desired_count: DESIRED_COUNT)
  end
end


# desc "delete service #{SERVICE}"
# task :delete do
#
#   aws ecs delete-service
#     --cluster CLUSTER
#     --service ${SERVICE_NAME}
#
# end


task :default do
  puts
  puts 'Use one of the available tasks:'
  system 'rake -T'
end
